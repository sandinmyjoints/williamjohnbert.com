---
comments: true
date: 2011-07-08 10:56:29
layout: post
slug: how-to-unit-testing-in-django-with-mocking-and-patching
title: 'How to: Unit testing in Django with mocking and patching'
wordpress_id: 346
categories:
- Web Development
tags:
- datetime
- django
- mocking
- python
- unit testing
---

### Background ###

For [Readsr](www.readsrs.com), I need to track events that recur on a particular day of the week (e.g., first Sunday of the month, third Friday of the month). I created a DayOfWeek model to store any particular event's day of the week. It contains a method next\_day\_of\_week() to return a datetime.date object set to the next occurrence of whatever weekday a given event instance is set to (this helps with figuring out when the next occurrence of an event is).

It's easier to show through an example. On Sunday 7/3/2011:

- For an object with DayOfWeek set to Sunday, next\_day\_of\_week() would return 7/3/2011 (current day).
- For DayOfWeek set to Monday, it would return 7/4/2011 (first subsequent Monday).
- For DayOfWeek set to Saturday, it would return 7/9/2011 (first subsequent Saturday).

Sounds simple enough. It seemed like this would be a good place to do my first unit tests.

### Unit Testing ###

To do unit testing, the typical method is to first write test cases and then write code. In this case, I'd already written my code, so I went back and wrote test cases, trying to forget how my code worked. 

To write test cases, you have to detail requirements for each method you want to test: input and expected (correct) output. The list of examples for 
next\_day\_of\_week() I wrote above works for this purpose. But there's a catch: next\_day\_of\_week() calculates the next day of the week relative to the current date, by calling datetime.date.today(). So if I write expected output for 7/3/2011, it will no longer be the correct output on 7/4/2011 or any following day. I needed a way to make datetime.date.today() always spit out my input date when I run tests, yet still continue to function normally outside of testing. Enter mocking.

### Mocking ###

The solution was to mock out the method--to replace the real datetime.date.today() with a fake one that produces the same output no matter what day it is. To accomplish this, I used the powerful [Mock library](http://www.voidspace.org.uk/python/mock/). Specifically, I needed to use the patch decorator. This decorator makes it really easy to replace on particular object within the scope of a particular method. 

Before I could patch the today() method, I needed to create my own fake method. It would look like this:


    
    
    def faketoday()
        return date(2011, 7, 4)
    



There's a problem, though, when I try to patch (or mock out) the method: 


    
    
    >>> import mock
    >>> def faketoday():
    ...     return datetime.date(2011, 7, 4)
    ... 
    >>> @mock.patch("datetime.date.today", faketoday)
    ... def testfunc():
    ...     return datetime.date.today()
    ... 
    >>> testfunc()
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
      File "/Users/wbert/.virtualenvs/readsr_env/lib/python2.6/site-packages/mock.py", line 561, in patched
        arg = patching.__enter__()
      File "/Users/wbert/.virtualenvs/readsr_env/lib/python2.6/site-packages/mock.py", line 623, in __enter__
        setattr(self.target, self.attribute, new_attr)
    TypeError: can't set attributes of built-in/extension type 'datetime.date'
    >>> 



datetime.date is considered a Python built-in and can't be modified.

### Modifying a Class That Can't Be Modified ###

The trick is to write a child class that can be modified, and thus faked:


    
    
    class FakeDate(date):
    	"A fake replacement for date that can be mocked for testing."
    	def __new__(cls, *args, **kwargs):
    		return date.__new__(date, *args, **kwargs)
    



All this does is create a class whose constructor returns an instance of its parent's class, date. Usually, this would be pointless, but it's useful here because the new class isn't a built-in and thus can be mocked.

To use it, we simply decorate any test method that calls datetime.date.today() with a patch to replace datetime.date with FakeDate, and we also provide FakeDate a fake today() method that returns only and always the particular we are going to use for testing:


    
    
    class TestDayOfWeek(TestCase):
    	"""Test the day of the week functions."""
    
    	@mock.patch('series.models.date', FakeDate)
    	def test_valid_my_next_day_of_week_sameday(self):
    		from datetime import date
    		FakeDate.today = classmethod(lambda cls: date(2011, 7, 3)) # July 3, 2011 is a Sunday
    		new_day_of_week = DayOfWeek.objects.create()
    		new_day_of_week.day = "SU"
    		self.assertEquals(new_day_of_week.my_next_day_of_week(), date(2011, 7, 3))



A couple things to note: the patch only applies within this particular method, so each method to use a patch must be decorated. Also, the real datetime.date is imported in the method so we can use it inside the fake today() method. We could put this fake today() method inside FakeClass, but making it a lambda (anonymous) method assigned inside the test case gives us the flexibility to set a particular date for each test case.

### Namespacing ###

You may be wondering why the patch decorator takes "series.models.date" as the method to replace instead of "datetime.date". That was how I tried it at first, and I was confused when it didn't work. It seemed as if the patch hadn't taken effect. 

Well, it hadn't. That's because within the module being tested (models.py in the series app, or series.models in Python dotted notation), date has been imported like so:


    
    
    from datetime import date
    



This means that within series.models, date is now available as series.models.date, so that's the name that needs to be mocked out. For more on namespacing when mocking, checked out [Mock's Where to patch documentation](http://www.voidspace.org.uk/python/mock/patch.html#id2).

Now we can supply out unit tests with any date we want, ensuring that we know what the results should be and can test against them. 

### References ###

Learning how to do this stuff, I posted [my first question at Stackoverflow](http://stackoverflow.com/questions/6575687/how-do-i-use-mocking-to-test-a-next-day-of-week-function) (I ended up answering it myself). I also learned about using a fake class [from this question](http://stackoverflow.com/questions/4481954/python-trying-to-mock-datetime-date-today-but-not-working). Finally, the [Mock documentation](http://www.voidspace.org.uk/python/mock/index.html) was very helpful as well.
