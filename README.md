# Conference Organization App API Project

### About

This project is a cloud-based API server to support a web-based and native Android application for conference organization.  The API supports the following functionality:

- User authentication (via Google accounts)
- User profiles
- Conference information
- Session information

The API is hosted on Google App Engine as application ID [conference-app-fullstack](https://conference-app-fullstack.appspot.com/#/), and can be accessed via the [API explorer](https://apis-explorer.appspot.com/apis-explorer/?base=https://conference-app-fullstack.appspot.com/_ah/api#p/).


### Useful resources
- [Udacity Git Hub](https://github.com/udacity/fullstack-nanodegree-conference/blob/master/models.py)
- [Google App Engine Python Docs](https://cloud.google.com/appengine/docs/python/)
- [Programming Google App Engine (Book)](http://www.amazon.com/Programming-Google-App-Engine-Sanderson/dp/144939826X)
- [Data Modeling for Google App Engine Using Python and ndb (Screencast)](https://www.youtube.com/watch?v=xZsxWn58pS0)
- [App Engine Modeling: Parent-Child Models (GitHub)](https://github.com/GoogleCloudPlatform/appengine-modeling-ndb/blob/master/parent_child_models.py)


### Design and Improvement Tasks

#### #1 Add Sessions to a Conference

Conference.py contains corresponding endpoints and methods.
I added the following endpoint methods in conferencepy

- `createSession`:    given a conference, creates a session.
- `getConferenceSessions`:   given a conference, returns all sessions.
- `getConferenceSessionsByType`:   given a conference and session type, returns all applicable sessions.
- `getSessionsBySpeaker`: given a speaker, returns all sessions across all conferences.

#### #2: Add Sessions to User Wishlist

I modified the `Profile` model to accommodate a 'wishlist' stored as a repeated key property field, named `sessionsToAttend`.  In order to interact with this model in the API, I also had modify some of the previous methods in Task 1 to return a unique web-safe key for sessions.  I added two endpoint methods to the API:

- `addSessionToWishlist`: given a session websafe key, saves a session to a user's wishlist.
- `getSessionsInWishlist`: return a user's wishlist.

#### #3: Indexes and Queries

I added two endpoint methods for additional queries that I thought would be useful for this application:

-`getConferenceSessionFeed`: returns a conference's sorted feed sessions occurring same day or later. This would be useful for users to see a chronologically sorted, upcoming "feed," similar to something like Meetup's feed.
-`getTBDSessions`: returns sessions missing time/date information. Many times, conferences will know the speakers who will attend, but don't necessarily know the time and date that they will speak. As an administrative task, this might be a useful query to pair with some background methods to automatically notify the creator to fill in the necessary data as the conference or session dates approach.

For the specialized query, finding non-workshop sessions before 7pm, I ran into the limitations with using ndb/Datastore queries.  Queries are only allowed to have one inequality filter, and it would cause a `BadRequestError` to filter on both `startDate` and `typeOfSession`.  As a result, a workaround I implemented was to first query sessions before 7pm with `ndb`, and then manually filter that list with native Python to remove sessions with a 'workshop' type.  This could have been done in reverse, and the query which would filter the most entities should be done with `ndb`.

#### #4: Add Featured Speaker

I modified the `createSession` endpoint to cross-check if the speaker appeared in any other of the conference's sessions.  If so, the speaker name and relevant session names were added to the memcache under the `featured_speaker` key.  I added a final endpoint, `getFeaturedSpeaker`, which would check the memcache for the featured speaker.  If empty, it would simply pull the next upcoming speaker.
