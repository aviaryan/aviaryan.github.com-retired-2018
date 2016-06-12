---
layout: post
title: Better fields and validation in Flask Restplus
category: gsoc
tags: gsoc gsoc16 flask flask-restplus python
---

We at [Open Event Server](https://github.com/fossasia/open-event-orga-server) project are using [flask-restplus](http://flask-restplus.readthedocs.io) for API. 
Apart from auto-generating of Swagger specification, another great plus point of restplus is how easily 
we can set input and output models and the same is automatically shown in Swagger UI. 
We can also auto-validate the input in POST/PUT requests to make sure that we get what we want.

{% highlight python %}
@api.expect(EVENT_POST, validate=True)
def put(self, id):
    """Modify object at id"""
    pass
{% endhighlight %}

As can be seen above, the `validate` param for `namespace.expect` decorator allows us to auto-validate the input payloads.
This used to work well until one day I realized there were a few problems. 

1. When a field was defined as say for example `field.Integer`, then it will accept only Integer values, not even `null`.
2. If there is a string field and it has `required` param set to True, then also it is possible to set empty string as its value and the in-built validator won't catch it.
3. Even if I somehow managed to [hack my way](https://github.com/noirbizarre/flask-restplus/issues/179#issuecomment-224544238) to support `null` in field, 
it will also support null even if required=True.
4. We had no control on what error message was returned.

{% highlight python %}
EVENT = api.model('Event', {
    'id': fields.Integer,
    'name': fields.String(required=True)
})
{% endhighlight %}

Specially problem #1 was a huge one as it questioned the whole foundation of the API.
So we realized it will be better if we don't use `namespace.expect` and use a custom validator. 
For custom validator, we first had to create custom fields that this validator can benefit from. Luckily flask-restplus comes with a great API for creating custom fields. 
So we quickly created custom fields for all common fields (Integer, String) and more specific fields like Email, Uri and Color. 
Creating these specific fields were a huge advantage as now we can show proper example for each field types in the Swagger UI.

{% highlight python %}
class Email(fields.String):
    """
    Email field
    """
    __schema_type__ = 'string'
    __schema_format__ = 'email'
    __schema_example__ = 'email@domain.com'
{% endhighlight %}

Consider the above code; now when we use `Email` as a field for a value, then the example shown for it in Swagger UI will be 'email@domain.com'. Quite cool, right?

Now we needed a way to validate these fields. For that, what we did was to create a `validate` method in each of the field-classes. 
This `validate` method would get the value and check if it was valid. Consider the following code - 

{% highlight python %}
import re
EMAIL_REGEX = re.compile(r'\S+@\S+\.\S+')

class Email():
	def validate(self, value):
		if not value:
		    return False if self.required else True
		if not EMAIL_REGEX.match(value):
		    return False
		return True
{% endhighlight %}

Once each of the field had their validate methods, we created a `validate_payload()` function that uses the API model and compares it with the payload. 
It will first check if all required keys are present in the payload or not. 
When that is true, it finally validates each field's value using their field's class `validate` method.

{% highlight python %}
from flask import abort
from flask_restplus import fields
from custom_fields import CustomField

def validate_payload(payload, api_model):
    # check if any reqd fields are missing in payload
    for key in api_model:
        if api_model[key].required and key not in payload:
            abort(400, 'Required field \'%s\' missing' % key)
    # check payload
    for key in payload:
        field = api_model[key]
        if isinstance(field, fields.List):
            field = field.container
            data = payload[key]
        else:
            data = [payload[key]]
        if isinstance(field, CustomField) and hasattr(field, 'validate'):
            for i in data:
                if not field.validate(i):
                    abort(400, 'Validation of \'%s\' field failed' % key)
{% endhighlight %}

The `CustomField` is the base class that each of the custom fields mentioned above inherit. So checking if `field` was an instance of `CustomField` is enough to know if it is 
a custom field or not. 
Other thing that may look weird in the above code is use of `fields.List`. If you look closely, I have added this to support custom fields inside lists. 
So if you have used a custom field in a list, it will also work too. But obviously, this only supports single level lists for now. 
The thing is we didn't needed more than that so I let it go. :stuck_out_tongue_winking_eye:

This basically sums up how we are validating input payloads at Open Event. Of course this is very basic but we will keep on improving it as the project progresses.
Stay tuned to [opev blog](http://opev.wordpress.com) if you want to be in touch with the progress of the project.

Links to full code at the time of writing this post are -

1. [Custom Fields](https://github.com/fossasia/open-event-orga-server/blob/2bb118147a56e6cfc7d3ed7a01d28efd2da6467b/open_event/api/custom_fields.py)
2. [Validate Payload](https://github.com/fossasia/open-event-orga-server/blob/2bb118147a56e6cfc7d3ed7a01d28efd2da6467b/open_event/api/helpers.py#L135)


I hope you found this post useful. Thanks for reading.
