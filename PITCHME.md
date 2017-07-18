# Generic foreign keys: why not?

---?image=images/heres-why-not.png&size=80%

---?image=images/hey-why-not.png&size=40%

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

### Generic foreign keys have this one weird trick

- ForeignKey can refer to one type of model
- GFKs can refer to any model in your application

---
### Let's keep track of changes to our models
```py 
class Event(models.Model):
    changes = JSONField()
    occurred_at = models.DateTimeField(auto_now_add=True)
    
    object_id = models.BigIntegerField()
    content_type = models.ForeignKey(ContentType)
    subject = GenericForeignKey('content_type', 'object_id')
```
@[1-3](We want to serialize the changes, and when they occurred)
@[5](We will store the ID of the model that changed)
@[6](We will store the type of the model that changed)
@[5-7](Wrap content_type and object_id as a GFK)
---
### What is GenericForeignKey doing?

```py
class GenericForeignKey(object):
    def contribute_to_class(self, cls, name, **kwargs):
        cls._meta.add_field(self, private=True)
        setattr(cls, name, self)
        
    def __get__(self, instance):
        return instance.content_type.get_object_for_this_type(
            instance.object_id
        )
```
@[2-4](Add self as a field on the model class)
@[6-9](Work as a descriptor using content type and object ID to return referenced model)

---
### Creation
```py
class Cleaner(models.Model):
    name = models.CharField(...)

class Job(models.Model):
    cleaner = models.ForeignKey(Cleaner)
    started_at = models.DateTimeField()
    
def assign_job(job, cleaner):
    job.cleaner = cleaner
    job.save()
    Event.objects.create(
        subject=job, changes={'cleaner': cleaner.id}
    )
    
def set_cleaner_name(cleaner, name):
    cleaner.name = name
    cleaner.save()
    Event.objects.create(
        subject=cleaner, changes={'name': name}
    )
```
---
### Retrieval
```py
>>> first_event = Event.objects.order_by('occurred_at').first()
>>> first_event.subject

<Job: cleaning at Pam's>

>>> last_event = Event.objects.order_by('occurred_at').last()
>>> last_event.subject

<Cleaner: Alex>
```
---
### Querying
```
>>> cleaner = Cleaner.objects.get(name='Alex')
>>> Event.objects.filter(subject=cleaner)

---------------------------------------------------------------------------
FieldError                                Traceback (most recent call last)
...
FieldError: Field 'subject' does not generate an automatic reverse relation...


>>> content_type = ContentType.objects.get_for_model(Cleaner)
>>> Spam.objects.filter(content_type=content_type, object_id=cleaner.id)]

<QuerySet [<Event: 'Name changed to Alex'>]>
```
---
### GenericRelations
---
## Generic foreign keys considered harmful?
Django core committers advising against using them

Luke Plant: <a href="https://lukeplant.me.uk/blog/posts/avoid-django-genericforeignkey/" target="_blank">Avoid Generic Foreign keys</a>.

Marc Tamlyn: <a href="https://www.youtube.com/watch?v=aDt4gu99_bE" target="_blank">Weird and wonderful things to do with the ORM</a>.


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
