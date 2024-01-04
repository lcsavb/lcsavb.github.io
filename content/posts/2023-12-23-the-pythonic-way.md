---
layout: post
title:  The Pythonic Way
date: 2024-01-04
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
#tags: [python, programming, web development]
author: Lucas Barros
---

Hello, everyone! Today, I want to talk about a technical aspect of my project and the evolution of my code.

The goal: to create a Django form responsible for making a new prescription.

And the problem is fairly simple: each prescription may have up to 4 drugs and each drug may have a different quantity and posology for each of 6 months.

So, the first step is to create a class which inherits from forms.Form:

```python
class CreatePrescription(forms.Form):
    form_field_1 = forms.CharField(required=True, label='Diagnóstico',widget=forms.TextInput)
    .
    .
    .
    .
    form_field_n = forms.CharField()


```

## The lazy approach

Initially, I used a manual approach, defining each field explicitly. This method was amateurish and did not adhere to the "DRY" (Don't Repeat Yourself) principle.

In portuguese that is what is called "sausage code". Does it work? Certainly! But adds a big "broken window" in the project.
And even a single broken window is more than enough to spread disarray throughout.

```python
class CreatePrescription(forms.Form):
    med1_posologia_mes1 = forms.CharField(required=True, label='Posologia')
    med1_posologia_mes2 = forms.CharField(required=True, label='Posologia')
    med1_posologia_mes3 = forms.CharField(required=True, label='Posologia')
    med1_posologia_mes4 = forms.CharField(required=True, label='Posologia')
    med1_posologia_mes5 = forms.CharField(required=True, label='Posologia')
    med1_posologia_mes6 = forms.CharField(required=True, label='Posologia')
    med2_posologia_mes1 = forms.CharField(required=False, label='Posologia')
    med2_posologia_mes2 = forms.CharField(required=False, label='Posologia')
    med2_posologia_mes3 = forms.CharField(required=False, label='Posologia')
    med2_posologia_mes4 = forms.CharField(required=False, label='Posologia')
    med2_posologia_mes5 = forms.CharField(required=False, label='Posologia')
    med2_posologia_mes6 = forms.CharField(required=False, label='Posologia')
    med3_posologia_mes1 = forms.CharField(required=False, label='Posologia')
    med3_posologia_mes2 = forms.CharField(required=False, label='Posologia')
    med3_posologia_mes3 = forms.CharField(required=False, label='Posologia')
    med3_posologia_mes4 = forms.CharField(required=False, label='Posologia')
    med3_posologia_mes5 = forms.CharField(required=False, label='Posologia')
    med3_posologia_mes6 = forms.CharField(required=False, label='Posologia')
    med4_posologia_mes1 = forms.CharField(required=False, label='Posologia')
    med4_posologia_mes2 = forms.CharField(required=False, label='Posologia')
    med4_posologia_mes3 = forms.CharField(required=False, label='Posologia')
    med4_posologia_mes4 = forms.CharField(required=False, label='Posologia')
    med4_posologia_mes5 = forms.CharField(required=False, label='Posologia')
    med4_posologia_mes6 = forms.CharField(required=False, label='Posologia')
    qtd_med1_mes1 = forms.CharField(required=True, label="Qtde. 1 mês")
    qtd_med1_mes2 = forms.CharField(required=True, label="Qtde. 2 mês")
    qtd_med1_mes3 = forms.CharField(required=True, label="Qtde. 3 mês")
    qtd_med1_mes4 = forms.CharField(required=True, label="Qtde. 4 mês")
    qtd_med1_mes5 = forms.CharField(required=True, label="Qtde. 5 mês")
    qtd_med1_mes6 = forms.CharField(required=True, label="Qtde. 6 mês")
    qtd_med2_mes1 = forms.CharField(required=False, label="Qtde. 1 mês")
    qtd_med2_mes2 = forms.CharField(required=False, label="Qtde. 2 mês")
    qtd_med2_mes3 = forms.CharField(required=False, label="Qtde. 3 mês")
    qtd_med2_mes4 = forms.CharField(required=False, label="Qtde. 4 mês")
    qtd_med2_mes5 = forms.CharField(required=False, label="Qtde. 5 mês")
    qtd_med2_mes6 = forms.CharField(required=False, label="Qtde. 6 mês")
    qtd_med3_mes1 = forms.CharField(required=False, label="Qtde. 1 mês")
    qtd_med3_mes2 = forms.CharField(required=False, label="Qtde. 2 mês")
    qtd_med3_mes3 = forms.CharField(required=False, label="Qtde. 3 mês")
    qtd_med3_mes4 = forms.CharField(required=False, label="Qtde. 4 mês")
    qtd_med3_mes5 = forms.CharField(required=False, label="Qtde. 5 mês")
    qtd_med3_mes6 = forms.CharField(required=False, label="Qtde. 6 mês")
    qtd_med4_mes1 = forms.CharField(required=False, label="Qtde. 1 mês")
    qtd_med4_mes2 = forms.CharField(required=False, label="Qtde. 2 mês")
    qtd_med4_mes3 = forms.CharField(required=False, label="Qtde. 3 mês")
    qtd_med4_mes4 = forms.CharField(required=False, label="Qtde. 4 mês")
    qtd_med4_mes5 = forms.CharField(required=False, label="Qtde. 5 mês")
    qtd_med4_mes6 = forms.CharField(required=False, label="Qtde. 6 mês")
```

## super()

The problem takes form: it is need to add the fields *dinamically*, the class __init__ method has to be overrided with super():

```python
class CreatePrescription(forms.Form):
    def __init__(self, *args, **kwargs):
        self.user = kwargs.pop('user', None)
        super(CreatePrescription, self).__init__(*args, **kwargs)
        # new approaches here

    # the rest of the form
    form_field_1 = forms.CharField(required=True, label='Diagnóstico',widget=forms.TextInput)
    .
    .
    form_field_n = forms.CharField()

```

The following approaches are then included in this new __init__ method.

## Approach 2: Loop-Based Creation

To improve, I moved to a loop-based approach, which made the code more dynamic and reduced repetition.

```python
class CreatePrescription(forms.Form):
    def __init__(self, *args, **kwargs):
        self.user = kwargs.pop('user', None)
        super(CreatePrescription, self).__init__(*args, **kwargs)

        drugs = ['01', '02', '03', '04']
        months = ['01', '02', '03', '04', '05', '06']

        for d in drugs:
            drug_field_name = f'drug_{d}'
            drug_field_label = f'Medicamento {d}'
            self.fields[drug_field_name] = forms.CharField(label=drug_field_label)

            for m in months:
                posology_field_name = f'posology_drug_{d}_month_{m}'
                posology_field_label = f'Posologia - Medicamento {d} - Mês {m}'
                qty_field_name = f'qty_drug_{d}_month_{m}'
                qty_field_label = f'Qtde. - Medicamento {d} - Mês {m}'
                self.fields[posology_field_name] = forms.CharField(label=posology_field_label)
                self.fields[qty_field_name] = forms.CharField(label=qty_field_label)
```

Certainly better, but not good enough.

## Approach 3: The zen of Python

```python

class CreatePrescription(forms.Form):
    def __init__(self, *args, **kwargs):
        self.user = kwargs.pop('user', None)
        super(CreatePrescription, self).__init__(*args, **kwargs)

        drugs = ['01', '02', '03', '04']
        months = ['01', '02', '03', '04', '05', '06']

        for d in drugs:
            drug_field_name = f'drug_{d}'
            drug_field_label = f'Medicamento {d}'
            self.fields[drug_field_name] = forms.CharField(label=drug_field_label)

        self.fields.update(
            {
                f'posology_drug_{d}_month_{m}': forms.CharField(label=f'Posologia - Medicamento {d} - Mês {m}')
                for d in drugs
                for m in months
            }
        )

        self.fields.update(
            {
                f'qty_drug_{d}_month_{m}': forms.CharField(label=f'Qtde. - Medicamento {d} - Mês {m}')
                for d in drugs
                for m in months
            }
        )
```

The definitive code is using the update method of the form's fields dictionary to add new fields dynamically. This is done inside a dictionary comprehension, which is a concise way to create dictionaries.

The update method takes a dictionary as an argument and adds its key-value pairs to the existing dictionary. If a key in the given dictionary already exists in the original dictionary, its value is updated.

In this case, the dictionary being added has keys in the format drug_{d} and values that are instances of forms.CharField. The {d} part is a placeholder that gets replaced with each item in the drugs iterable. The label argument to forms.CharField is set to Medicamento {d}, which means each field will have a label like "Medicamento 1", "Medicamento 2", etc., depending on the values in drugs.

This is a common pattern in Django when you need to add fields to a form dynamically based on some variable data. In this case, the form fields are being created based on the drugs iterable.

*And it is important to highlight again: the __init__ class has to be overrided with super()!*

## Conclusion

Comparing these three approaches, the list comprehension with `update` method stands out as the best. It’s not only more efficient and easier to maintain, but also scales well with future changes and additions. This method is ideal for regular, dynamic data structures like the example of form fields for medicines and posology.

---
