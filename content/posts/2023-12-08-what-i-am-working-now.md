---
layout: post
title:  A webapp to facilitate prescription 
date: 2023-12-08
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
#tags: [books, test]
author: Lucas
---


Hello, everyone! Today, I want to share with you a project I've been working on for some time, which is very close to my heart - AutoCusto. This project is a solution I developed to tackle the papework needed for the High Cost Pharmacies (Farmácias de Alto Custo).

## My first attempt to solve it

The first time I tackled this issue was in late 2018. Since I primarily had to fill prescriptions for 5 or 6 diseases and I had no clue how to code whatsoever, I manually compiled all the forms specific to each prescription into a single PDF file. To accomplish that I bought the pro version of Adobe Acrobat.

This allowed me to print them with just one click. I assigned the same label to all repetitive fields, allowing me to enter each piece of repetitive information just once. For instance, the patient's name needed to be included on every sheet. By labeling each name field as "PATIENT NAME" I ensured that entering the name on the first sheet automatically filled it in on all subsequent sheets.

For each patient, I created a new file from the model, saved it, and simply updated the date whenever I needed to renew the prescription.

However, about 6 months later because the government changed the standard format of the forms. So hundreds of saved templates and the time I invested in building them were rendered worthless.

## My Solution: AutoCusto

It was in this context that AutoCusto was born. The name is a play on words in Portuguese, combining "Alto" (High cost) with "Auto" (Automatic). The goal of AutoCusto is to minimize errors, rework, and inconvenience for patients. The primary focus of a doctor should be patient care, not navigating unnecessary bureaucracy. The very first version was a CLI (Command Line Interface) program, but soon I evolved it into a fully-fledged web app written in Python.

The alpha version served its purpose for my personal use, although it was unsuitable for production due to issues that I will address in subsequent posts.

## Cost-benefit analysis

### Quantitative

I worked from December 2019 to August 2020 for approximately 3 hours a day, 5 days a week, which amounted to 600 hours of development. This period includes all the time I spent learning everything from scratch.

I had 400 regular patients. As some of them used more than one drug, that ammounted to 500 prescriptions which had to be repeated every six months. Taking into consideration the previous ammount of error, which ranged in the 20%'s figure, let's consider 600 prescriptions every semester, the time to fill each one of then felt from 20 minutes to 3 minutes.

**Therefore, the ammount of time saved per year was 300 hours (15 minutes * 600 prescriptions * 2 / 60).**

The alpha version was ready in March 2020 and I have used until June 2023. Three and a half years. That saved 1050 hours of my life, a **net gain of 450 hours**.

So, let's also consider the time saved by the patients. Considering 200 wrong prescriptions per year, a 1 hour commute by public transport to the pharmacy and 2 hours waiting in line, that is 600 hours of time saved.

| Description                                | Details                                                  |
|--------------------------------------------|----------------------------------------------------------|
| **Development Period**                     | December 2019 to August 2020                             |
| **Use of Alpha Version**        | March 2020 to June 2023 (3.5 years)                      |
| **Hours Spent in Development**             | Approximately 600 hours                                  |
| **Regular Patient Count**                  | 400                                                      |
| **Prescriptions per Semester**             | 600 (including 20% error margin)                         |
| **Time Reduction per Prescription**        | From 20 minutes to 3 minutes                             |
| **Time Savings per Prescription**          | ~15 minutes                                               |
| **Time Savings per Year**              | 300 hours                                                |
| **Economic Analysis**                      |                                                          |
| - **Doctor's Hourly Wage in Brazil**         | €40                                                      |
| - **Annual Savings**            | €12,000 (150 hours x €40 x 2 semesters)                  |
| - **Total Hours Saved by Doctor**          | 1050 hours                                               |
| - **Total savings**  | €42,000                                                  |
| - **Net Gain of Hours**           | 450 hours                                                |
| **Patient Time Savings Analysis**          |                                                          |
| - **Incorrect Prescriptions per Year**     | 200                                                      |
| - **Commute Time to Pharmacy**             | 1 hour (by public transport)                             |
| - **Waiting Time in Pharmacy**             | 2 hours                                                  |
| - **Total Annual Time Saved for Patients** | 600 hours per year                                       |
| - **Total Time Saved for Patients**        | 2100 hours (600 hours x 3.5 years)                       |

### Qualitative

Out of the 450 net hours saved, I spent most of them caring for patients. I also dedicated a significant amount of time to reading articles, drinking coffee, and playing with dogs in the square adjacent to the clinic where I worked. This software also provided peace of mind to my patients, knowing they could drop by the clinic anytime they needed (without an appointment, which might not be available in the short term), and I could quickly redo their prescriptions if necessary. 

Yet my most valuable personal gain was the study I had to undertake! I picked up Python, Django, and Docker, gained familiarity with JavaScript for front end develpment, along with HTML. Additionally, I delved into courses like [CS50](https://www.youtube.com/channel/UCcabW7890RKJzL968QWEykA), [Introduction to Computer Science with Python - OCW](https://ocw.mit.edu/courses/6-0001-introduction-to-computer-science-and-programming-in-python-fall-2016/) and a couple more. [Lucas Ricciardi](https://github.com/LucasRicciardi), an accomplished developer and teacher, deserves special acknowledgment, as he taught me the fundamentals of Python and pointed me where to follow afterwards. 

Ultimately, I discovered that coding is just one aspect; deploying and commercializing software presents an entirely different challenge – a truly important lesson. I have not deployed yet. Currently there is a similar software called [LME FÁCIL](http://lmefacil.com.br) (accessible only through a VPN - the ip has to be in Brazil) which has about 70.000 registered patients and works well. I will explore its caveats to make AutoCusto even better.

The depth and impact of this learning journey are immeasurable in mere numbers, as it has profoundly changed the trajectory of my life - that is why I am here now at the Deggendorf Institute of Technology.

## The future

I am rewriting AutoCusto from scratch, correcting all the beginner's mistakes. My current plan, which is subject to change, is to make it available as open source and offer consulting services to any company or governmental branch interested in using it. From my perspective, the ideal application would be to have an administrative worker handle the prescriptions, with the doctor only needing to review and sign them afterwards.

Recently, the Brazilian Health Ministry approved the acceptance of prescriptions with a cryptographic signature. This is an absolute must-have feature that I will incorporate. Since the pandemic, the use of cryptographically signed prescriptions has become widespread in Brazil.

Finally, I am going to create a free API. This is particularly important because there currently isn't an open API available for the specific data of high-cost pharmacies. This information will be helpful for the development of other healthcare solutions.