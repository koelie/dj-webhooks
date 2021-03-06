=============================
dj-webhooks
=============================

.. image:: https://img.shields.io/pypi/dm/dj-webhooks.svg
        :target: https://pypi.python.org/pypi/dj-webhooks

.. image:: https://badge.fury.io/py/dj-webhooks.png
    :target: https://badge.fury.io/py/dj-webhooks

.. image:: https://img.shields.io/pypi/wheel/dj-webhooks.svg
    :target: https://pypi.python.org/pypi/dj-webhooks/
    :alt: Wheel Status

.. image:: https://travis-ci.org/pydanny/dj-webhooks.png?branch=master
    :target: https://travis-ci.org/pydanny/dj-webhooks

Django + Webhooks Made Easy

The full documentation is at https://dj-webhooks.readthedocs.org.

Requirements
------------

* Python 2.7.x or 3.3.2 or higher
* django>=1.5.5
* django-jsonfield>=0.9.12
* django-model-utils>=2.0.2
* django-rq>=0.6.1
* webhooks>=0.3.1

Quickstart
----------

Install dj-webhooks::

    pip install dj-webhooks

Configure some webhook events:

.. code-block:: python

    # settings.py
    WEBHOOK_EVENTS = (
        "purchase.paid",
        "purchase.refunded",
        "purchase.fulfilled"
    )

Add some webhook targets:

.. code-block:: python

    from django.contrib.auth import get_user_model
    User = get_user_model()
    user = User.objects.get(username="pydanny")

    from webhooks.models import Webhook
    WebhookTarget.objects.create(
        owner=user,
        event="purchase.paid",
        target_url="https://mystorefront.com/webhooks/",
        identifier="User or system defined string",
        header_content_type=Webhook.CONTENT_TYPE_JSON,
    )

Then use it in a project:

.. code-block:: python

    from django.contrib.auth import get_user_model
    User = get_user_model()
    user = User.objects.get(username="pydanny")

    from djwebhooks.decorators import hook

    from myproject.models import Purchase

    # Event argument helps identify the webhook target
    @hook(event="purchase.paid")
    def send_purchase_confirmation(purchase, owner, identifier): 
        return {
            "order_num": purchase.order_num,
            "date": purchase.confirm_date,
            "line_items": [x.sku for x in purchase.lineitem_set.filter(inventory__gt=0)]
        }

    for purchase in Purchase.objects.filter(status="paid"):
        send_purchase_confirmation(
            purchase=purchase, 
            owner=user,
            identifier="User or system defined string"
        )

Storing Redis delivery logs
---------------------------

**Note:** The only difference between this and the previous example is the use of the redislog_hook.

.. code-block:: python

    from django.contrib.auth import get_user_model
    User = get_user_model()
    user = User.objects.get(username="pydanny")

    from djwebhooks.decorators import redislog_hook

    from myproject.models import Purchase

    # Event argument helps identify the webhook target
    @redislog_hook(event="purchase.paid")
    def send_purchase_confirmation(purchase, owner, identifier): 
        return {
            "order_num": purchase.order_num,
            "date": purchase.confirm_date,
            "line_items": [x.sku for x in purchase.lineitem_set.filter(inventory__gt=0)]
        }

    for purchase in Purchase.objects.filter(status="paid"):
        send_purchase_confirmation(
            purchase=purchase, 
            owner=user,
            identifier="User or system defined string"
        )


In a queue using django-rq
----------------------------

**Warning:** In practice I've found it's much more realistic to use the ORM or Redislib webhooks and define seperate asynchronous jobs then to rely on the ``djwebhooks.redisq_hook decorator``. Therefore, this functionality is deprecated.



Features
--------

* Synchronous webhooks
* Delivery tracking via Django ORM.
* Options for asynchronous webhooks.

Planned Features
-----------------

* Delivery tracking via Redis and other write-fast datastores.
