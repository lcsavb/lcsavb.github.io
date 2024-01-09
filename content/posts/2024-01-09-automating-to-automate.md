---
layout: post
title:  Automating to automate
date: 2024-01-09
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
#tags: [python, programming, web development]
author: Lucas Barros
---


# How did I create my data?

The SUS (Brazilian Unified Health System) provides a limited number of medications though it's "High Cost Pharmacies" program. Back in 2019 when I started this project, there wasn't a public database or API to the data - and to my knowledge there is not one until this very day.

The data was all spread out in the site of the Health Ministry and in guidelines in PDFs.

So, I had two choices: either add the data and link manually or...

## Automating to automate

All the code I used to accomplish this is located in the repository: [Data Retrieval](https://github.com/lcsavb/autocusto-data-retrieval).

It was five years ago and I have learned two important lessons: to never again use bad words* in my code as I have
done and, MOST IMPORTANTLY, be organized and write documentation. It is a mess and I don't remember quite well all
the steps I followed, but to the best of my ability I'll try to make a summary.

*but you don't need to translate them.... or eat peanut butter either.

## Data

The data structure is pretty straightforward.

Protocols for specific diseases, yet each protocol has multiples ICDs after all there are different numbers for
the same disease. For example, the for the Epilepsy Protocol one can choose G40.0 or G40.1. The total number of protocols is 92 which are vinculated to about 500 ICD codes.

For each protocol there are an unique set of conditional documents and a set of authorized drugs. That is it.

I was able to vinculate and organize all the data, with the exception of the conditional documents.



### Getting the protocols

The first step was to download the PDFs corresponding to each protocol:

```python
import urllib.request
from bs4 import BeautifulSoup
from tika import parser
import requests
import substring
import os

url_indice = 'http://www.saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/relacao-estadual-de-medicamentos-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-protocolo-clinico-e-diretriz-terapeutica'
indice = urllib.request.urlopen(url_indice)

sopa = BeautifulSoup(indice, 'html.parser')

for link in sopa.find_all('a'):
    endereco = str(link.get('href'))
    if '/resources/' in endereco:
        if endereco[0] != 'h':
            endereco = 'http://saude.sp.gov.br' + endereco
        endereco = str(endereco)
        os.system('wget ' + endereco)
```

### Diving into PDFs

Next I used an PDF crawler to search through the PDFs and generate the following JSON file.
That way I was able to vinculate the Protocol with the ICDs. As you see it, this is a raw file and I
was able to get the ICD codes because every groups of ICDs in these files was inside parenthesis. I accomplished
to separete each one because every single one starts with an Uppercase letter followed by 2 numbers.
But as you see it, I had to fix some things: I have used an open ICD API to normalize the values and corrected
manually the few wrong which where left.

'''python

import glob
import os
from tika import parser
import substring
import json

arquivos = glob.iglob('/home/lucas/dev/chupador/medicamentos/vinculador/protocolos/*.*')

protocolos_cids = {}
for a in arquivos:
    texto = parser.from_file(a)['content']
    lista_cids = busca_cid(texto)
    try:
        nome_protocolo = substring.substringByChar(str(a), '_', '.')
        nome_protocolo = nome_protocolo[1:-1]
        nome_arquivo = os.path.basename(a)
    except:
        nome_protocolo = 'nao identificado'
        nome_arquivo = os.path.basename(a)

    protocolos_cids.update({nome_protocolo: {'cids': lista_cids, 'arquivo': nome_arquivo}})

protocolos_json = json.dumps(protocolos_cids, indent=4, sort_keys=True)

with open('protocolos.json', 'w') as novo_arquivo:
    novo_arquivo.write(protocolos_json)

'''one thing
            "G30.0",
            "G30.1",
            "G30.8",
            "F00.0",
            "F00.1",
            "F00.2",
            "v11.p",
            "v10_2",
            "v10_2",
            "B12;",
            "B12;"
        ]
    },
    "doencadecrohnv9": {
        "arquivo": "20_doencadecrohnv9.pdf",
        "cids": [
            "K50.0",
            "K50.1",
            "K50.8",
            "v17.p",
            "v17.p"
        ]'''
        "arquivo": "21_doencadegaucherv11.pdf",
        "cids": [
            "E75.2",
            "v12.p",'''python
'''

## How about the drugs?

Next, I had to know which drug is vinculated with each protocol. I could have done it with the PDFs, yet
extracting the drug's names from them was too tricky so I used a shortcut. I realized that every single drug
was in the website's menus of the Health Secretary, so using a web crawler again I got a raw file with all the contents of the website. For example:

```
['Clique ', <a href="http://saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-a'''ssistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/medicamentos-para-tratamento-de-glaucoma">aqui</a>, ' para orientações']
['\xa0']
['Secretaria de Estado da Saúde']
['Av. Dr. Enéas Carvalho de Aguiar,188 - São Paulo - Fone (11) 3066 8000 - CEP 05403-000']
[]
['No Estado de São Paulo, o\xa0acesso aos medicamentos para tratamento de glaucoma se dá em:']
['\xa0']
['Pacientes atendidos em um dos Serviços de Referência em Oftalmologia, habilitados pelo SUS, abaixo relacionados:\xa0clique ', <a href="http://saude.sp.gov.br/resources/ses/perfil/cidadao/acesso-rapido/medicamentos/relacao-estadual-de-medicamentos-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-protocolo-clinico-e-diretriz-terapeutica/39b_glaucoma_v3_-_servicos_de_referencia.pdf">aqui</a>, '\xa0para orientações.']
['Pacientes atendidos por outros serviços de saúde:\xa0clique ', <a href="http://saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/medicamentos-para-tratamento-de-glaucoma">aqui</a>, '\xa0para orientações.']
['\xa0']
```

After that I used a python script to retrieve only the links.

```html

    "acitretina": [
        "http://www.saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/acitretina"
    ],
    "adalimumabe": [
        "http://www.saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/adalimumabe"
    ],
```

Luckily for me, inside every link was the ICDs associated with the respective drug. I extracted the ICDs using the same method already mentioned.

At that point I know which protocol and which is vinculated with each ICD. But not which drugs are vinculated with each protocol.

But the drugs have diferent presentations and dosages. After getting the raw data ([https://github.com/lcsavb/autocusto-data-retrieval/tree/master/medicamentos/csv_raw](https://github.com/lcsavb/autocusto-data-retrieval/tree/master/medicamentos/csv_raw)/tree/master/medicamentos/csv_raw) I crossed it with a drug API to normalize it.

## Vinculating drugs with the protocols

So, cross-referencing the cids I found in the website with the cids in the pdfs I was able to finally associate all the drugs with the protocols with minimal error.

## The end

Was it the best way? Well, I am not really sure. But I know for sure: it was pretty faster than doing t all by hand, I learned a lot of Python and the important lesson to be more organized and thoroughly with the documentation!

So, in broad strokes that was what I did it to get the data to start my project. Lastly I wrote some python
code to populate the database and correct the remaining mistakes in the data. These scripts can be seen in:
https://github.com/lcsavb/autocusto/tree/master/processos/db.

Thank you for reading.
