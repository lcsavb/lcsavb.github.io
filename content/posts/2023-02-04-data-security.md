---
layout: post
title:  Enhancing Data Security
date: 2024-02-04
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
#tags: [python, programming, web development]
author: Lucas Barros
---

Safeguarding sensitive data is paramount, especially when dealing with personal information such as patients' data in a healthcare application. The Django web framework offers robust tools for building secure applications, but it's crucial to apply best practices to enhance data protection and comply with privacy regulations. 

### The Initial Code

The initial code snippet provided was a Django view function designed to search for patients based on a keyword that could match either a patient's name or CPF (a Brazilian tax identification number). Here's the original code:

```python
from django.http import JsonResponse
from django.db.models import Q
from .models import Patient

def search_patients(request):
    keyword = request.GET.get('keyword', None)
    queryset = Patient.objects.filter(Q(patient_cpf__icontains=keyword) |
                                      Q(patient_name__icontains=keyword))
    patients = []
    for patient in queryset:
        new_patient = {'patient_name': patient.patient_name, 'patient_cpf': patient.patient_cpf}
        patients.append(new_patient)

    return JsonResponse(patients, safe=False)
```

While functional, the initial code posed several security and privacy concerns:

1. **Exposure of Sensitive Data**: Considering the ManyToMany relationship between users and patients, the initial code did not restrict access based on this relationship, allowing users to access data for patients they are not associated with.
2. **Lack of Authentication and Authorization**: There was no mechanism in place to ensure that only authenticated and authorized users could access the data.

### Enhanced Version

To address these concerns, the code was refactored with the following enhancements:

```python
from django.http import JsonResponse
from django.db.models import Q
from .models import Patient
from django.contrib.auth.decorators import login_required

@login_required
def search_patients(request):
    keyword = request.GET.get('keyword', '').strip()
    if not keyword:
        return JsonResponse({'error': 'Keyword not provided'}, status=400)

    # Access only to patients associated with the logged-in user
    associated_patients = request.user.patients.all()

    # Filter associated patients by name or CPF
    filtered_patients = associated_patients.filter(
        Q(patient_name__icontains=keyword) | Q(patient_cpf__icontains=keyword)
    ).values('patient_name', 'patient_cpf', 'id')

    patients_list = list(filtered_patients)

    return JsonResponse(patients_list, safe=False)
```

### Key Improvements

1. **Authentication**: The `@login_required` decorator was added to ensure that only authenticated users could access the patient search functionality.
2. **Authorization**: The revised code assumes a many-to-many relationship between users and patients, ensuring users can only access data for patients they are associated with.

### One last thing

But, instead of writing

```python
associated_patients = request.user.patients.all() 
``` 

in every funcion it would be best to code a decorator to wrap them:

```python
from functools import wraps
from django.http import JsonResponse

def check_patient_association(view_func):
    @wraps(view_func)
    def _wrapped_view(request, *args, **kwargs):
        # Filter only the patients associated with the logged-in user
        associated_patients = request.user.patients.all()

        # Add the associated patients to 'request' for use in the view
        request.associated_patients = associated_patients
        return view_func(request, *args, **kwargs)

    return _wrapped_view



@login_required
@check_patient_association
def search_patients(request):
    keyword = request.GET.get('keyword', '').strip()
    if not keyword:
        return JsonResponse({'error': 'Keyword not provided'}, status=400)

    # Use the associated patients added by the decorator
    associated_patients = getattr(request, 'associated_patients', None)

    # Filter the associated patients by name or CPF based on the keyword
    filtered_patients = associated_patients.filter(
        Q(patient_name__icontains=keyword) | Q(patient_cpf__icontains=keyword)
    ).values('patient_name', 'patient_cpf', 'id')

    patients_list = list(filtered_patients)

    return JsonResponse(patients_list, safe=False)


```

In summary, the `check_patient_association` decorator is used to add associated patients to the `request` object before a view function is executed. This allows the view function to focus on the view logic, while the decorator handles fetching the associated patients.

### Conclusion

Enhancing the security and privacy of web applications is an ongoing process that requires attention to detail. In the case of the Django application discussed, applying authentication and refining authorization checks significantly improved the security posture. The goal is to insure that data is accessible only to those with a legitimate need and adequately protected against unauthorized access.





---
