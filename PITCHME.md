# Generic foreign keys: why not?

---?image=images/hey-why-not.png&size=40%

---?image=images/heres-why-not.png&size=80%

---
### Foreign key
- Reference that uniquely identifies a model
- Implemented at database level
- Firm guarantees of referential integrity
- Automatically indexed for performance
- Effortless querying forward and backwards

---
### Generic foreign key
- Reference that uniquely identifies a model
- ~~Implemented at database level~~
- ~~Firm guarantees of referential integrity~~
- ~~Automatically indexed for performance~~
- ~~Effortless querying forward and backwards~~

---?image=images/scarcely.png

---
## Generic foreign keys considered harmful?
Django core committers advising against using them

Luke Plant: <a href="https://lukeplant.me.uk/blog/posts/avoid-django-genericforeignkey/" target="_blank">Avoid Generic Foreign keys</a>.

Marc Tamlyn: <a href="https://www.youtube.com/watch?v=aDt4gu99_bE" target="_blank">Weird and wonderful things to do with the ORM</a>.

---
## This one neat trick

GFKs can refer to any model in your application.

There are some great applications for such a trick.

---
## Applications: adding metadata to any model
- Revisions
- Tags
- Common properties, such as telephone numbers

---
### Applications: adding to any model
- Comments
- Tickets
- Ratings

---
### Applications: containing many different things
- Tumble log entry
  - Can contain a link, a video, an image etc
  
---
### Applications: portable third party apps
- You run an email service, and you want to provide a Django app
- Create your own model to record a sent email
- Add GFKs for sent_to and sent_by
  - Any model in the user's app can send to any other model
  
---
Used properly, can enable:
- Looser coupling
- Extensability
- Openness about the future

---
## So there are these great applications. Can we overcome the bad bits?
---
### Definition
```py
class Customer(models.Model):
    email = models.EmailField()
   
class Employee(models.Model):
    email = models.EmailField()
    
class Spam(models.Model):
    text = models.TextField()
    sent_at = models.DateTimeField(auto_now_add=True)
    
    object_id = models.BigIntegerField()
    content_type = models.ForeignKey(ContentType)
    sent_to = GenericForeignKey('content_type', 'instance_id')
```
---
### Content Types
- Uniquely identifies a Django model
---
### Creation
```py
def send_and_record_spam(instance):
    spam_text = generate_spam()
    send_email(instance.email, spam_text)
    Spam.objects.create(sent_to=instance, text=spam_text)
    
def spam_customers():
    for _ in xrange(100000):
        send_and_record_spam(Customer.objects.order_by('?').first())

def spam_employees():
    for _ in xrange(100000):
        send_and_record_spam(Employee.objects.order_by('?').first())
```
---
### Retrieval
```py
>>> first_spam = Spam.objects.order_by('sent_at').first()
>>> first_spam.sent_to

<Customer: Pam>

>>> last_spam = Spam.objects.order_by('sent_at').last()
>>> last_spam.sent_to

<Employee: Raj>
```
---
### Querying
```
>>> customer = Customer.objects.get(email='spam_trap@gmail.com')
>>> Spam.objects.filter(sent_to=customer)

---------------------------------------------------------------------------
FieldError                                Traceback (most recent call last)
...
FieldError: Field 'sent_to' does not generate an automatic reverse relation and therefore cannot be used for reverse querying. If it is a GenericForeignKey, consider adding a GenericRelation.

>>> content_type = ContentType.objects.get_for_model(Customer)
>>> Spam.objects.filter(content_type=content_type, object_id=customer.id)]

<QuerySet [<Spam: 'You have won the lottery'>, <Spam: 'You have won the lottery'>, <Spam: 'You have won the lottery'>, ... ]>
```
