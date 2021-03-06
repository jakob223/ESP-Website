Program Modules
===============

.. contents:: :local:

The Good Parts
--------------

Program modules are an ESP website-specific framework for bits of code associated to a program.  Like many parts of the ESP website, they do a lot of really useful things, but they’re also a bit complicated, arguably more complicated than necessary.  They serve roughly the following functions:

* organize code
* allow certain bits of code to be activated only for certain programs
* easily create views which are specific to a program and work in a certain way
* direct students and teachers through a set of steps, some optional and some required, for a program’s registration
* aggregate data from various parts of the website into the dashboard
* have various utilities for writing views

If you’re writing a new view for a program, you probably want to put it in a program module, either an existing one or a new one.  Program modules live in ``/esp/esp/program/modules/handlers/``.  They consist of a python file, for example ``foobarmodule.py``, containing a class, for example ``FooBarModule`` which inherits from ``esp.program.modules.base.ProgramModuleObj``.  (Note that other utilities mentioned in this section live in the same file.)  For the most part, you can just copy existing program modules and figure out how they work.  If you want to know all the details of what can go in them, read the next section.  For a list of the program modules, and what they do, see ``/docs/admin/program_modules.rst``.

The Dirty Details
-----------------
Your program module should have:

* A docstring, because you love your fellow devs and want them to be happy.
* A ``classmethod`` ``module_properties(cls)`` which returns a dict with the following keys, or a list of such dicts with different ``module_types``.

  * ``admin_title``: the title that will appear in the admin panel
  * ``link_title``: the title that will appear in the manage page, student registration, or teacher registration
  * ``module_type``: manage, teach, learn, or onsite (or json, but you probably won’t need that)
  * ``seq`` (optional): the default sequence index of the module, for sorting it in various lists
  * ``required`` (optional, default False): True if the student/teacher should by default be required to complete the module as a part of registration
  * ``class Meta: proxy = True`` (this is a Django thing that tells it not to create a new database table specifically for instances of your module)
  * Optionally, a method ``isCompleted(self)`` that returns a boolean to figure out whether the user has completed the module (e.g. filled out the medical form)
  * Optionally, a method ``students(self, QObject=False)`` and ``studentDesc(self)``, which return dicts where each key maps to a ``QuerySet`` and a string describing them, respectively, to be added to the list of student stats on the dashboard.  If ``QObject=True``, the method should return a dict of ``Q`` objects instead.  The corresponding methods for teachers may also be included.

It will then (optionally) have one method with the decorator ``@main_call``, and optionally one or more methods with the decorator ``@aux_call``.  (It can have other methods, too; they are not handled specially.)  These are the views of the program module; they behave somewhat like django views, with the following caveats:

* They always take the arguments ``self``, ``request``, ``tl``, ``one``, ``two``, ``module``, ``extra``, and ``prog``.  You will probably never need to use any of the arguments except ``self``, ``request``, ``extra``, and ``prog``.
* They will automatically appear at ``/<module_type>/<program>/<instance>/<methodname>``.  If the URL additionally has ``/<something>`` at the end, ``<something>`` will be passed to the view as ``extra``.
* The method with decorator ``@main_call`` will be linked in the appropriate list: the “complete list of modules” on the manage page, the student or teacher registration checklist, or the onsite landing page.
* They usually use templates from ``self.basedir() + whatever.html``, which is ``/templates/program/modules/<modulename>/whatever.html``.
* There are a number of useful decorators that can be added to them, underneath the ``@main_call`` or ``@aux_call``:

  * ``@needs_{student,teacher,admin}``: Only allow this type of user to use the module; others will get an error telling them they need to be this type of user.  Should go above any ``@meets_grade`` or ``@meets_deadline`` decorators so they get this error first.
  * ``@meets_grade``: only allow students in the grade range for this program.
  * ``@meets_deadline()``: Only allow users with the given permission type (which will automatically have Student or Teacher prepended as appropriate); give others a deadline error.
