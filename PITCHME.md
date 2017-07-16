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
