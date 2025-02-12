## Upgrading Your Opal Application

This document provides instructions for specific steps required to upgrading your Opal
application to a later version where there are extra steps required.

#### v0.15.0 -> v0.16.0
Diagnosis.details is now a text field. Please make and run migrations if you are
using it in your app.

The record template for diagnosis now shows the details on the next row.


### 0.13.1 -> 0.14.0
If you were creating a UserProfile, this will now fail as a UserProfile is created automatically after creating a user (by a signal).


### 0.13.0 -> 0.13.1

Upgrades django (minor security upgrade).

How you do this depends on how you have configured your application. You will need to
update both the Opal version, and versions of upgraded dependencies if
you have specified them in for instance, a requirements.txt.

(This will be the case if you use the requirements.txt originally provided by
`opal startproject`)

    # requirements.txt
    opal==0.13.1
    django==2.0.13


### 0.12.0 - 0.11.2 -> 0.13.0

#### Python versions

Opal 0.13.0 drops support for Python 2.x
If you have not already done so, you will need to upgrade your application to Python 3
in order to upgrade.

You may also like to run the tests for your application with the 'show warnings'
flag e.g.  `python -Wd manage.py test`

#### Upgrading Opal

How you do this depends on how you have configured your application. You will need to
update both the Opal version, and versions of upgraded dependencies if
you have specified them in for instance, a requirements.txt.

(This will be the case if you use the requirements.txt originally provided by
`opal startproject`)

    # requirements.txt
    opal==0.13.0
    django==2.0.9
    django-reversion==3.0.1
    djangorestframework==3.7.4
    letter==0.5
    psycopg==2.7.6.1
    python-dateutil==2.7.5
    requests=2.20.1

#### Free text or foreign key fields are now, by default case insensitive

It is recommended you resave all model values for fk_or_ft fields as this will give you
consistent behaviour. Otherwise fk_ft values wihch differ from fkt values only by
case prior to this upgrade will be stored as ft and those afterwards will be stored as
the relevant fk.


#### Migrations

You will need to run the migrations for Opal 0.13.0

    $ python manage.py migrate opal

As Opal 0.13.0 contains changes to the definition of lookuplists, you will
need to run a makemigrations command to update your lookuplists to enable code
values and change case sensitivity.

    python manage.py makemigrations yourapp
    python manage.py migrate yourapp


### 0.11.1 -> 0.11.2

This bugfix release should be entirely backwards compatible.

### 0.11.0 -> 0.11.1

This bugfix release should be entirely backwards compatible.

### 0.10.1 -> 0.11.0

Please upgrade django-compressor version to 2.2, ie update your requirements to

##### requirements.txt

django-compressor==2.2

### 0.10.0 -> 0.10.1

#### Upgrading Opal

How you do this depends on how you have configured your application, but updating your
requirements.txt to update the version should work.

    # requirements.txt
    opal==0.10.1

There are no migrations or additional commands for this upgrae, and we are not aware of
any backwards incompatible changes.

### 0.9.0 -> 0.10.0

#### Upgrading Opal

How you do this depends on how you have configured your application. You will need to
update both the Opal version, and versions of dependencies upgraded dependencies if
you have specified them in for instance, a requirements.txt.

(This will be the case if you use the requirements.txt originally provided by
`opal startproject`)

    # requirements.txt
    django==1.10.8
    django-reversion==1.10.2
    djangorestframework==3.4.7
    psycopg==2.7
    ffs==0.0.8.2
    opal==0.10.0
    requests==2.18.4
    django-celery==3.2.2
    celery==3.1.25
    django-compressor==2.2


After re-installing (via for instance `pip install -r requirements.txt`) you will
likely need to make some changes to your application code to fix errors that arise as
a result of upstream breaking changes.

##### Breaking changes from dependencies

The jump in Django versions introduces some breaking changes. Some common problems are
outlined here.

*Early imports of models*

Importing models in a package __init__.py file is no longer allowed - if you see
`django.core.exceptions.AppRegistryNotReady` this is the likely cause.

*Django reversion*

Reversion has reorganized its package structure to avoid importing models.
Specifically:

```python
# Old-style import for accessing the admin class.
import reversion
# New-style import for accesssing admin class.
from reversion.admin import VersionAdmin

# Old-style import for accessing the low-level API.
import reversion
# New-style import for accesssing the low-level API.
from reversion import revisions as reversion
```

You should examine your `admin.py` to see whether you do this.

*Urlpatterns*

`django.conf.urls.patterns` is now removed rather than deprecated. The default urlconfs
generated by older Opal versions use them, but these should now be replaced with lists.

*TEMPLATES setting*

You will need to convert to the new style TEMPLATES-* dictionary-style configuration in settings.py. Details are in the [relevant section of the Django documentation](https://docs.djangoproject.com/en/1.10/ref/templates/upgrading/#the-templates-settings). Note that Django's own context processors have moved from `django.core` to `django.template`.

*AUTOCOMPLETE_SEARCH setting*

The (previously undocumented) setting `AUTOCOMPLETE_SEARCH` has been renamed to
`OPAL_AUTOCOMPLETE_SEARCH` to match other Opal settings. Update your `settings.py`
to the new setting name if you had enabled this feature.

##### Migrations

You will need to run the migrations for Opal 0.10.0

    $ python manage.py migrate opal

As Opal 0.10.0 contains changes to the cascading deletion behaviour of subrecords, you will
need to run a makemigrations command to update your subrecords.

    python manage.py makemigrations yourapp
    python manage.py migrate yourapp


##### LoginRequiredMixin

Django now ships with `django.contrib.auth.mixins.LoginRequiredMixin`. Accordingly we have
removed `opal.core.views.LoginRequiredMixin`. A direct switch to the Django class should
work seamlessly without any functional differences.

##### CSRF_FAILURE_VIEW
We now ship the `opal.views.csrf_failure` view which can be enabled by adding
`CSRF_FAILURE_VIEW = 'opal.views.csrf_failure'` in your settings.py. This will
redirect a user to their intended destination on a CSRF failure. This mitigates
an edge case where an unauthenticated user opens two pages at the same time.
Both pages will get redirected to the login form and whichever page the user
logs into second will throw a CSRF failure because Django invalidates CSRF
tokens on login.


### 0.8.3 -> 0.9.0

`episode.date_of_episode`, `episode.date_of_admission` and `episode.discharge_date` are all deprecated.

We now expect episodes to use `episode.start` and `episode.end`. You should search your codebase for any
instances where the three variables are used and switch to the start/end properties.

### 0.8.2 -> 0.8.3
No changes.

### 0.8.1 -> 0.8.2

The application menu API, previously python dicts stored in attributes on either plugin or
application subclasses, now consists of the new `opal.core.menus.MenuItem` class, enabling
enhanced customisation and flexibility.

If you previously were adding Menu Items with the dictionary format. These should be
converted to use a `MenuItem` class.

### 0.8.0 -> 0.8.1

#### Upgrading Opal

How you do this depends on how you have configured your application, but updating your
requirements.txt to update the version should work.

    # requirements.txt
    opal==0.8.0

There are no Opal migrations for this version.

#### PatientList.get_queryset

The `PatientList.get_queryset` method now takes keyword arguments - at the current time
it is only passed `user` - the user for the current request, but in the future this may
be extended, so applications should consider using `**kwargs` to ensure this method
does not raise exceptions in the future.

#### ReopenEpisodeCtrl

We've removed the undocumented `ReopenEpisodeCtrl` from Opal. Applications wishing
to retain this functionality may copy the controller from the v0.7.1 branch
(opal/static/js/opal/controllers/reopen_episode.js). However we would warn developers
that this flow has proven to be problematic and confusing for users whenever
used - hence the removal.

### 0.7.1 -> 0.8.0

#### Upgrading Opal

How you do this depends on how you have configured your application, but updating your
requirements.txt to update the version should work.

    # requirements.txt
    opal==0.8.0

After re-installing (via for instance `pip install -r requirements.txt`) you will need to
run the migrations for Opal 0.6.x

    $ python manage.py migrate opal

#### Options

Options are now an ex-API. Applications should convert to use either Referencedata (canonical terms for common data), or
Metadata (App specific data you wish to pass into the front end).

#### UI Components

Those applications relying on Angular strap ui components (Typeahead, Popover,
Tooltip, Datepicker, Timepicker) should convert their templates to use the Angular UI Boostrap equivalents, or the
Opal templatetags.

If you are simply using Opal templatetags from `forms` and not overriding these templates, then the transition should be seamless. Otherwise, searching your codebase for html files containing `bs-` and looking for angular strap components is a good start.

Applications or plugins with javascript tests may need to update their includes to remove references to old library files.

Full documentation of the markup and options for these components is found [here](http://angular-ui.github.io/bootstrap/versioned-docs/0.14.3/)

#### extending modal_base.html

We now have different base templates for modals, forms and two column modal forms (essentially a form with a side bar).

The form templates add validation checks around the saving to catch any validation errors a form might through. They assume the existence of a form called 'form'.

As part of this modal_base has been moved into a folder in templates called base_templates

Rename any templates extending `modal_base.html` to extend the correct template in `base_templates/`. This will be either `modal_base.html` or `modal_form_base.html`.

#### Add episode modal url

The add episode modal previously available at
`/templates/modals/add_episode.html/` is now not available at the url with a trailing slash.
Any controllers attempting to open the modal e.g. custom list flows should update their
`$modal.open` call to remove the trailing slash.

#### Admin URL

The Admin url has changed to require a trailing slash, so any links to `/admin` - for instance, in a custom menu bar, will need to be updated to `/admin/`.

### 0.7.0 -> 0.7.1

#### Downstream dependencies

Opal 0.7.1 updates the expected version of Django Axes to 1.7.0 - you will wish to update
this in your requirements.txt or similar accordingly.

#### DRF Authentication

We highly recommend that applications explicitly set Django Rest Framework authentication
classes in their `settings.py`.

By default Opal now uses session and token auth, which will require a migration to install
the DRF Token authentication app.

```python
INSTALLED_APPS = (
    # ....
    'rest_framework',
    'rest_framework.authtoken',
    # ...
)

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    )
}
```

### 6.x -> 7.x

#### Upgrading Opal

How you do this depends on how you have configured your application, but updating your
requirements.txt to update the version should work.

    # requirements.txt
    opal==0.7.0

After re-installing (via for instance `pip install -r requirements.txt`) you will need to
run the migrations for Opal 0.6.x

    $ python manage.py migrate opal


If you are inheriting from the abstract models in Opal e.g. `Demographics` then you should
run a makemigrations command to update to the 0.7.x data model.

    python manage.py makemigrations yourapp
    python manage.py migrate yourapp


#### Breaking changes

Opal 0.7 contains a number of breaking changes.

##### Name changes

`opal.models.Episode.category` has been re-named `category_name`. If your application
directly sets category, you will be required to update all instances where this happens.

The `/episode/:pk/` API has moved to `/api/v0.1/episode/:pk/` so any code (typically
javascript) code that directly saves to this API endpoint rather than using the Opal JS
`Episode` services should work immediately when re-pointed at the new URL.

##### Moving from options to referencedata and metadata

The signature of the EditItemCtrl has been updated - this modal controller no longer
takes an `options` argument, rather it uses the new 0.7.x `referencedata` and `metadata`
services. Applications that call EditItemCtrl directly should look to update the `resolves`
option they were passing to `$modal.open`. (Alternatively, developers should consider
refactoring to use the new `recordEditor` API.)

The signatures of Flow `enter` and `exit` methods has changed to no longer accept
options as a positional argument, and enter/exit controllers will no longer be initialized
with access to options as a resolved provider. They will have access to either/both of
`referencedata` and `metadata` so if your application includes custom flow controllers that
use `options` you will need to refactor these to use the new x-data arguments instead.

`referencedata` and `metadata` between them have all data previously in options, so the refactor
here should be relatively painless.

##### Date of birth fields in forms

The partial `partials/_date_of_birth_field.html` has been removed and replaced with the
`{% date_of_birth_field %}` templatetag in the forms library. You should update any forms
to use this new tag.

### 5.x -> 6.x

#### Upgrading Opal

How you do this depends on how you have configured your application, but updating your
requirements.txt to update the version should work.

    # requirements.txt
    opal==0.6.0

After re-installing (via for instance `pip install -r requirements.txt`) you will need to
run the migrations for Opal 0.6.x

    $ python manage.py migrate opal

#### Changes to abstract models

If you are inheriting from the abstract models in Opal e.g. `Demographics` then you should
run a makemigrations command to update to the 0.6.x data model.

    python manage.py makemigrations yourapp
    python manage.py migrate yourapp

You should note that as of Opal 0.6.x `Demographics` now splits names into first, surname,
middle name and title. The previous `name` field will be converted to be `first_name`.

Strategies for updating your data to use the appropriate fields will vary from application
to application, but one good such strategy is to use a data migration [such as the one done
here](https://github.com/openhealthcare/acute/blob/master/acute/migrations/0004_auto_20160624_1215.py).

#### Update settings

Many of the default Opal templates now assume that the `'opal.context_processors.models'`
Context Processor is available - you should add that to the `TEMPLATE_CONTEXT_PROCESSORS`
setting in your application's `settings.py`

The default date formats in Opal have changed - and so you should update your `DATE_X`
settings to match:

```python
DATE_FORMAT = 'd/m/Y'
DATE_INPUT_FORMATS = ['%d/%m/%Y']
DATETIME_FORMAT = 'd/m/Y H:i:s'
DATETIME_INPUT_FORMATS = ['%d/%m/%Y %H:%M:%S']
```

#### Upgrade plugins

A number of Opal plugins have new releases to work with the changes in Opal 0.6.x

* opal-referral - Upgrade to 0.1.4
* opal-wardround - Upgrade to 0.6.0
* opal-observations - Upgrade to 0.1.2
* opal-dischargesummary - Upgrade to 0.2.0
* opal-dashboard - Upgrade to 0.1.3

Meanwhile the `opal-taskrunner` plugin has now been deprecated, this functionality now
living natively within Opal core.

#### Update your Teams to be PatientLists

Patient Lists are now driven by subclasses of `opal.core.PatientList`, so we will need
to convert your Teams to be PatientLists. You may want to re-enable the Team admin while
you do so - this is simple, by updating your application's `admin.py`:

    # yourapp/admin.py
    ...
    from opal.admin import TeamAdmin
    from opal.models import Team
    admin.site.register(Team, TeamAdmin)


Patient lists are now declarative. For instance, to replicate the following team:

<img src="/img/resp.team.png" style="margin: 12px auto; border: 1px solid black;"/>


We would convert that to:

```python
# yourapp/patient*lists.py
from opal.core import patient_lists

class RespiratoryList(patient_lists.TaggedPatientList):
    display_name = 'Respiratory'
    tag          = 'respiratory'
    order        = 4
    schema       = [models.Demographics, models.Treatment]
```

The schema property will likely be available to you in your application's `schema.py`
file - which is now obsolete.

See the [full patient list documentation](../guides/list_views.md) for further details
of the options available for Patient Lists.

#### Form and Display templates.

We may now be missing some form or display templates, as your application may be
relying on templates previously in Opal. To discover which these are, run

    $ opal scaffold --dry-run

You may either create templates by hand, or have Opal generate boilerplate templates for you
by running `$ opal scaffold`.

Modal templates already in your application will likely be referencing invalid paths
to their Angular variables. You should update these to include the record name - for example:

```html
<!-- Was -->
{% input  label="Drug" model="editing.drug" lookuplist="antimicrobial_list" %}
<!-- Becomes -->
{% input  label="Drug" model="editing.treatment.drug" lookuplist="antimicrobial_list" %}
```

#### The Inpatient episode category

The default Episode Category - Inpatient episodes has updated it's database identifier
from `inpatient` to `Inpatient`. To update your episodes run :

```python
>>> from opal.models import Episode
>>> for e in Episode.objects.filter(category='inpatient'):
...   e.category='Inpatient'
...   e.save()
...
```

Any references to episode category in templates (for e.g. ng-hide) or controllers for logic
will also require updates.

#### Flow is now defined in JS

Flow is no longer defined on the server side in python, but rather is a javascript service.
See the documentation for information about setting up custom flows. At a minimum applications
that use custom flows will have to implement their own flow service and reference it in their
settings.

### 4.X -> 5.x

#### Migrations

Before upgrading from 4.x to 5.x you should ensure that you have upgraded from South
to Djangomigrations.

    $ rm yourapp/migrations/*
    $ python manage.py makemigrations yourapp
    $ python manage.py migrate yourapp --fake-initial

#### Opal

Next you will need to upgrade the Opal version itself.

How you do this depends on how you have configured your application, but updating your
requirements.txt to update the version should work. This will also update FFS and Django
Axes as well as adding Python Dateutil.

    -e git://github.com/openhealthcare/opal.git@v0.5.6#egg=opal


#### Migrations.

Opal has fresh migrations in 0.5.x, which we should run. There are also changes to the
base abstract model classes (to add created/updated timestamps) so you'll need to create
fresh migrations for your own application.

    $ python manage.py migrate
    $ python manage.py makemigrations yourapp
    $ python manage.py migrate yourapp

At this stage you'll want to commit your new migrations, as well as any changes to your
application's requirements file.

#### Tags

As of 0.5.5, old tags in Opal are stored directly on the Tagging model rather than via
Django Reversion. We can import those old tags by doing the following.

    $ python manage.py shell

    >>> from opal.models import Tagging
    >>> Tagging.import_from_reversion()

#### Deployment

The first time you deploy your upgraded application you'll need to run the following
commands to upgrade your database:

    $ python manage.py migrate --fake-initial

You'll also have to repeat the Tagging step once for each deployment.
