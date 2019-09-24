---
layout: post
title: "Rapidly prototyping GUIs in Python using Traits and TraitsUI"
date: 2019-08-20
---

My internship at Enthought is coming to a close, so I thought I'd do a post about the kind
of stuff I've been working on there. It's interesting that in a world of web-based apps,
some companies are still making traditional desktop applications. Building GUIs for either
the web or desktop are an excellent application of design patterns, with some of the earliest
software patterns in the 1980s arising from the need to separate complex application logic
from changeable and lightweight views of data.

These days a there are a ton of 'reactive' frameworks for web-based applications. Think React,
Angular, Vue.js etc. However, there is basically nothing similar for the desktop. Probably
because so few companies still work on this stuff anymore. However, as Enthought does for most
of its consultancy, they've built a lightweight Python reactive framework for GUIs, TraitsUI.
In this post I'm going to demonstrate some of its capabilities.

Static type checking has become a thing as of Py35, however Enthought has been developing in
Python for decades, involved in the language from way before the development of the modern
Python ecosystem in 2001. Therefore they built their own, and frankly much more useful, type
checking system in Python called Traits. To declare a Traits capable class all you need to do
is:


```python
from traits.api import (
    HasTraits, Int, Str
)


class Student(HasTraits):
    name = Str('srinath')
    age = Int(25)
    subject = Str('Computer Science')
```

As you can see it's already removed a ton of boiler plate code that Python forces you
to write to specify what happens upon object instantiation, and let you easily specify
default values for attributes. What makes this even better is that these attributes
are actually also reactive components, which means that you can setup code to listen
to changes in them. Perfect for GUI programming where a user is interacting with data.

```python
from traits.api import (
    HasTraits, Instance, Int, on_trait_change, Str 
)


class Professor(HasTraits):
    supervisee = Instance(Student)
    name = Str("A Professor")	
    faculty = Str('Mathematics') 	

    @on_trait_change('student:age')
    def update_student_details(self, new_age):
    self.student.age = new_age


class Student(HasTraits):
    name = Str('srinath')
    age = Int(25)
    subject = Str('Computer Science')


if __name__ == "__main__":
    student = Student()
    professor = Professor(student=student)
    student.age = 26
    print(professor.student.age)  # 25
```

This simple decorator allows you to listen to changes in attributes of another object.
I've snuck in some extra stuff here, like custom attributes (via the `Instance`) trait.
But the basics should be easy to follow. Now we can put these together with TraitsUI to
create a GUI essentially for free.

TraitsUI knows how to handle these HasTraits classes, so all you have to do is tell it
how to view your data (or 'model').


```python
from traitsui.api import (
    Item, ModelView, View
)


class UI(ModelView):
    model = Instance(Professor)
    
    view = View(
        Item('model.student.name', label='Student'),
	Item('model.faculty', label='Faculty'),
)


if __name__ == "__main__":
    ui = UI(model=professor)
    ui.configure_traits()
```

Running the above code you should see a fully fledged GUI pop up, albeit just
showing couple of components. This was all in less than 50 lines of code.

There is a little bit of magic in the above commands, and I defer to the docs
for more info (even though they are frankly quite sparse and difficult to use)
unfortunately, this tech was never picked up by the Python community at large,
despite how useful and clean (and fast) it is to write statically typed classes,
with the extra UI functionality. So, often, the source code is your best friend,
and even then it can be a complicated read... This is challenging software to get
your head around, but if you get the 'hang' of it, its a convenient way to rapidly
create little GUIs.
