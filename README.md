# django-compositepk-model

Provide an extended Django Model class named 'CPkModel' that supports composite primary keys. Also provide an extended Query class 'CPkQuery' that supports multi-column lookups.

This package supports treating legacy db tables with a composite primary key, without adding a surrogate key.

This is my other approach to [ticket373](https://code.djangoproject.com/ticket/373).

## Features

### 1. Easy to use

For a table with a composite primary key, change the base class from Model to CPkModel, and set 'unique_together' in Meta. In addition, set 'primary_key=True' to each primary keys.

(model definition sample)

```python
from django.db import models
from cpkmodel import CPkModel

# Normal Model
#   primary_key is auto 'id'
class Company(models.Model):
    name = models.CharField(max_length=100)
    established_date = models.DateField()
    company_code = models.CharField(max_length=100)

    class Meta:
        db_table = 'Company'

# Child Model (CpkModel)
#   composite primary key: company_id, country_code
class CompanyBranch(CPkModel):
    company = models.ForeignKey(
        Company,
        primary_key=True,       # for CompositePK
        on_delete=models.CASCADE)
    country_code = models.CharField(
        max_length=100,
        primary_key=True,       # for CompositePK
    )
    name = models.CharField(max_length=100)
    established_date = models.DateField()

    class Meta:
        db_table = 'CompanyBranch'
        unique_together = (('company', 'country_code'),)  # for CompositePK
```

That's all. No additional definitions or virtual fields are required.

### 2. Admin avairable

CPkModel can be used in Django Admin. The values of composite primary key are displayed comma separated style. Change(Update), Delete are fine. Add(Create) has a problem that CreateView do unique check to each key Field. So you can't add enough child records. But, this is only CreateView's problem. Your program can create child records by QuerySet or Model method.

### 3. Multi-column lookup avairable

To support composite primary keys, CPkQuery supports multi-column lookups.

```python
obj = CompanyBranch.objects.get(pk=(1,'JP'))
qs = CompanyBranch.objects.filter(pk='1,JP')
qs = CompanyBranch.objects.filter(**{'company,country_code':(1,'JP')})
qs = CompanyBranch.objects.filter(**{'company_id,country_code':'1,JP'})
```

LHS with comma is not just primary, also other columns are OK.

```python
qs = CompanyBranch.objects.filter(**{'country_code,name':'JP,Google'})
```

Lookup Multi-Column with 'in' is also avairable. PostgreSQL is fine, But SQLite3 is not supported.

```python
qs = CompanyBranch.objects.filter(pk__in=[(1,'JP'),(1,'US'),(2,'JP'),])
qs = CompanyBranch.objects.filter(**{'country_code,name__in':[('JP','HONDA'),('CN','SONY'),]})
```

## Installation

pip install django-compositepk-model

## Links

https://code.djangoproject.com/ticket/373  
https://code.djangoproject.com/wiki/MultipleColumnPrimaryKeys  
https://gijutsu.com/en/2021/01/19/django-composite-primary-key/  
