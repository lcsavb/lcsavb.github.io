---
layout: post
title:  Automatically retrieving data
date: 2024-01-09
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
#tags: [python, programming, web development]
author: Lucas Barros
---


# The challenges

The SUS (Brazilian Unified Health System) provides a limited number of medications though it's "High Cost Pharmacies" program. Back in 2019 when I started this project, there wasn't a public database or API to the data - and to my knowledge there is not one until this very day.

The data was all spread out in the site of the Health Ministry and in PDFs.

So, I had 3 choices:

1) Collect everything manually, yet **the total number of protocols is 92 which are vinculated to about 500 ICD codes and 400 to 700 drugs.** Not wise.

2) Do not collect any data at all and let the forms free for filling without constraints. But that would beat the purpose of reducing mistakes.

or

3) **Let the code do the heavy-lifting** - which actually became an automation in itself - a mini-project inside my project.


## Data

The data structure is pretty straightforward:

1) A Protocol consists of a single disease represented by many ICDs.

2) Each Protocol has many drugs associated with it.

For example:
- "Epilepsy Protocol": 
ICDs: G40.0, G40.1, G40.2, G40.3, G40.4, G40.5, G40.6, G40.7, G40.8.
Drugs: Lamotrigine, Clobazam, Levetiracetam, etc.

### Diving into PDFs

An example for the PDFs for each protocol can be seen [here](https://github.com/lcsavb/autocusto/tree/master/static/protocolos).

First I had to download them and the links for every single one were contained in a HTML page:

```python
import urllib.request
from bs4 import BeautifulSoup
import requests
import os

url_with_pdfs = # a long and ugly url lyied here

url = urllib.request.urlopen(url_with_pdfs)
        ''' links to the pdfs always started with a number and
        all the other links in that website of an H from https '''

        if link[0] != 'h': 
            link = 'http://saude.sp.gov.br' + link
        link = str(link)

        os.system('wget ' + link)
```

The next step was to write a function which would allow me to extract the ICDs associated with each protocol.

```python

def search_icd(file_content):
        ''' This function retrieves the ICDs in the pdf file,
        It does this by iterating over all the contents and 
        checking if each character is an uppercase letter followed 
        by two digits. If it is, it takes a slice of five characters 
        from the string, removes trailing whitespace, 
        and appends it to the list.
 
            '''
        possible_icds_list = []
        character = c

        for c in range(len(file_content)):
                if file_content[c].isupper() and file_content[c].isalpha():
                        if file_content[c + 1].isdigit() and file_content[c + 2].isdigit():
                                possible_icd = (file_content[c:c+5]).rstrip()
                                possible_icds_list.append(possible_icd)
        
        return possible_icds_list
```
OK! That worked, but there is a much elegant way to perform the same task using regular expressions.

```python
import re

possible_icds_list = re.findall(r'[A-Z]{2}\d{2}\.\d', file_content)
```

Next that function is used with each downloaded file.

```python

import glob
import os
from tika import parser
import substring
import json

# Get a list of all files in the specified directory
directory = # here was an ugly directory path
files = glob.iglob( string(directory) + '*.*')

# Initialize an empty dictionary to store the protocols and their ICDs
protocols_icds = {}

# Loop over each file
for file in files:
    # Parse the file content
    content_from_pdf = parser.from_file(file)['content']
    # Search for ICDs in the file content
    icd_list = search_icd(content_from_pdf)
    try:
        # Extract the protocol name from the file name
        # the output from the file name 20_PROTOCOL_NAME_v9.pdf will be PROTOCOL_NAME
        protocol_name = substring.substringByChar(str(file), '_', '.')
        protocol_name = protocol_name[1:-1]
        # Get the base name of the file
        file_name = os.path.basename(file)
    except:
        # If an error occurs, set the protocol name to 'not identified'
        protocol_name = 'not identified'
        file_name = os.path.basename(file)

    # Update the dictionary with the protocol name, ICDs, and file name
    protocols_icds.update({protocol_name: {'icds': icd_list, 'file': file_name}})

# Convert the dictionary to a JSON string
protocols_json = json.dumps(protocols_icds, indent=4, sort_keys=True)

# Write the JSON string to a new file
with open('protocols.json', 'w') as new_file:
    new_file.write(protocols_json)
```

That way I was able to vinculate the Protocol with the ICDs. Notice that ICD codes always starts with an uppercase letter followed by 2 numbers.

Here's an excerpt of the generated [JSON file](https://github.com/lcsavb/autocusto-data-retrieval/blob/master/medicamentos/vinculador/protocolos.json):

```json
    "doencadecrohnv9": {
        "file": "20_doencadecrohnv9.pdf",
        "icds": [
            "K50.0",
            "K50.1",
            "K50.8",
            "v17.p",
            "v17.p"
        ]
```

But as you can see, I had to tackle some rough edges: V17.p or v12.p are not ICDs. That happened because in the original version I made a little mistake with the search icd function. Nevertheless, I used an ICD API to normalize the values and manually corrected the few errors that were left.

## How about the drugs?

Next, I needed to determine which drug was associated with each protocol. I considered using the PDFs, but extracting the drug names proved too challenging due to the lack of uniformity or pattern in their naming, which a parser could not reliably identify.

So I found a shortcut! I realized that every single drug was listed in the Health Secretary's website menus, so by using a web crawler again, I obtained a raw file with all the contents of the website. For example:

```
['Clique ', <a href="http://saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-a'''ssistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/medicamentos-para-tratamento-de-glaucoma">aqui</a>, ' para orientações']
['\xa0']
['Secretaria de Estado da Saúde']
['Av. Dr. Enéas Carvalho de Aguiar,188 - São Paulo - Fone (11) 3066 8000 - CEP 05403-000']
[]
['No Estado de São Paulo, o\xa0acesso aos medicamentos para tratamento de glaucoma se dá em:']
['\xa0']
['Pacientes atendidos em um dos Serviços de Referência em Oftalmologia, habilitados pelo SUS, abaixo relacionados:\xa0clique ', <a href="http://saude.sp.gov.br/resources/ses/perfil/cidadao/acesso-rapido/medicamentos/relacao-estadual-de-medicamentos-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-protocolo-clinico-e-diretriz-terapeutica/39b_glaucoma_v3_-_servicos_de_referencia.pdf">aqui</a>, '\xa0para orientações.']
['Pacientes atendidos por outros serviços de saúde:\I accomplished
to separete each one because every single one starts with an Uppercase letter followed by 2 numbers.
    if '/resources/' in endereco:xa0clique ', <a href="http://saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/medicamentos-para-tratamento-de-glaucoma">aqui</a>, '\xa0para orientações.']
['\xa0']
```

After that, I utilized a Python script to filter the links, extract the drug names, associate both and save them into another JSON file.

```json

    "acitretina": [
        "http://www.saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/acitretina"
    ],
    "adalimumabe": [
        "http://www.saude.sp.gov.br/ses/perfil/gestor/assistencia-farmaceutica/medicamentos-dos-componentes-da-assistencia-farmaceutica/links-do-componente-especializado-da-assistencia-farmaceutica/consulta-por-medicamento/adalimumabe"
    ],
```

Luckily for me, inside every link was the ICDs associated with the respective drug. I extracted the ICDs using the same method already mentioned.

At that point, I knew which protocol and drug were associated with each ICD. By cross-referencing these ICDs (those extracted from the PDFs with those from the website), I was able to finally associate all the drugs with their respective protocols with minimal error.

Moreover the drugs have diferent presentations and dosages. After getting the [raw data](https://github.com/lcsavb/autocusto-data-retrieval/tree/master/medicamentos/csv_raw) I crossed it with a drug API to normalize it.

## Final thoughts

It was much faster - and certainly less boring - than doing it all by hand and I learned a lot about Python and the important lesson of being more organized and thorough with documentation!

It took me about 10 hours to code everything. Had I done it manually, considering 10 minutes per medication and 500 different drugs, it would have taken me 80 hours. A net savings of approximately 70 hours.

So, in broad strokes, that was what I did to gather the data needed to start my project. Lastly, I wrote some Python code to populate the database and correct the remaining errors in the data. These scripts can be found at: https://github.com/lcsavb/autocusto/tree/master/processos/db.

## A piece of advice to my fellow students

All the code I used to accomplish this is located in the repository: [Data Retrieval](https://github.com/lcsavb/autocusto-data-retrieval). It involved a little more steps than I have just described.

The repo's is messy: it basically involves lack of documentation and variables's names not only hard to decipher, but in portuguese.

That is a testament to my then naivety. The reasoning was twofold: I never intended to share it with anyone, and I treated the data-collecting as a one-off endeavor: it would only be run once. 

However, I've since realized that this mindset was flawed. As a beginner in programming, it’s essential to cultivate and reinforce good habits from the start. Effective software maintenance and comprehension over the long term demand adherence to best practices. Had I done that, writing this blog post would've been much easier!

Life's journey is unpredictable, often leading us to unexpected destinations, much like where I find myself today.