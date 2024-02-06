---
layout: post
title:  Refactoring code 2 - Security and Front End
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

## Using a global variable in Django to solve the problem 

The first approach would be to create a global variable ```associated_patients```  in the settings.py file and use the following code: 

```python

from django.http import JsonResponse
from django.db.models import Q
from django.contrib.auth.decorators import login_required
from settings import associated_patients

@login_required
def search_patients(request):
    keyword = request.GET.get('keyword', '').strip()
    if not keyword:
        return JsonResponse({'error': 'Keyword not provided'}, status=400)

    # Fetch associated patients from the global variable
    user_id = request.user.id
    if user_id in associated_patients:
        associated_patients_for_user = associated_patients[user_id]
    else:
        associated_patients_for_user = []

    # Filter the associated patients by name or CPF based on the keyword
    filtered_patients = filter(
        lambda patient: keyword.lower() in patient['patient_name'].lower() or
                        keyword.lower() in patient['patient_cpf'].lower(),
        associated_patients_for_user
    )

    patients_list = list(filtered_patients)

    return JsonResponse(patients_list, safe=False)

```

However, there are some downsides: 

- Global variables are not thread-safe, so if your Django application runs in a multi-threaded environment (which it typically does), concurrent modifications to the global variable can lead to race conditions and unpredictable behavior.
- Storing large amounts of data in a global variable can impact memory usage and performance.
- Global variables should be used sparingly and with caution, especially in web applications where concurrent access is common.

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

Enhancing the security and privacy of web applications is an ongoing process that requires attention to detail. In the case of the Django application discussed, applying authentication and refining authorization checks significantly improved the security posture. The goal is to insure that data is accessible only to those with a legitimate need and adequately protected against unauthorized access.


# Working with Django and JavaScript

What is the problem with the following JS code? It is designed to dinamically manipulate a dropdown field for selecting an issuer for a medical prescription.

``` javascript
    $.ajax({
        url: 'http://www.url.com' 
        method: 'GET',
        dataType: 'json',
        success: function(data) {
            issuerSelect.empty();
            issuerSelect.append($('<option></option>').attr('value', '')
            .text('Selecione o emissor').prop('disabled', true)
            .prop('selected', true));
            $.each(data.issuers, function(index, issuer) {
                issuerSelect.append($('<option></option>')
                .attr('value', issuer.id).text(issuer.name));
            });
        }
    });
```

It performs an AJAX GET request, expecting a JSON response. Upon successful response, the code first clears the issuerSelect dropdown, then adds a default option "Select the issuer" that is disabled and selected. Subsequently, it iterates over the issuers array in the JSON response, appending each medical prescription issuer as an option to the issuerSelect dropdown, using the issuer's id for the option value and the issuer's name for the displayed text.

The issue is the **hardcoding of the url**. Could getting the url dinamically from Django be a solution?

``` javascript
url: {% url 'issuers' %} 
```

That does not work! Django template tags are processed on the server side before the page is sent to the browser. JavaScript files, executed on the client side, cannot interpret Django template tags since these tags have already been rendered or resolved by the server when the page is delivered to the client.

To circumvent the limitation of embedding Django-specific logic within JavaScript, a common workaround is to use HTML data- attributes. This approach involves setting a data- attribute in an HTML element with the dynamically generated URL using the Django {% url %} tag. The URL is then accessed from the JavaScript by querying the data- attribute of the HTML element. 

``` javascript
url: issuerSelect.data('url');
```
And the html:

``` html
<select id="issuer-select" data-url="{% url 'get-issuers' %}"></select>
```

This method maintains code clarity, eases maintenance, and adheres to the separation of concerns principle by keeping server-side routing logic and client-side DOM manipulation logic well-separated.

---
