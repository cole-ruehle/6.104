# Problem Set 1: Concept Design

## Exercise 1: Reading a concept

1. **Invariants**: 

  *There are at least as many items as there are Requests*

  *Each Request has >1 purchase*

1. **Fixing actions**:

   *Since every pruchase must be attached to an existing request, if you remove an item that has a purchase and it gets deleted that rule is broken*

2. **Inferring behavior**: 

   *Yes it can open an close repeatidly this could be useful in the case that they need to edit the registry while it could be in use, or if you want to make sure it is not open during events/only open during events*

3. **Registry deletion**:

   *This would not matter in practice because a closed registry that is never opened is essentially deleated. This is how many applications do it anyways.*

4. **Queries**: 

   *addItem(Registry 1, shoes, 2) -- By the owner*
   *purchase(Jeff, Registry 1, shoes, 1)*

5. **Hiding purchases**:

   *A naïve way to do that would be to to overload the add item so that the user can do it as well*
   *A Better solution would be to add a new action of adding a secret item or giver item that does not have to be added by the user.*

6. **Generic types**:

   *Codes or ids are shorter and can be made to not repeat. Representing with price is not descriptive enough of the item.*


---

## Exercise 2: Extending a concept

**concept** PasswordAuthentication

**purpose** limit access to known users

**principle** after a user registers with a username and a password,
  they can authenticate with that same username and password
  and be treated each time as the same user

**state**
  a set of Users with
      a username String
      a password String

**actions**
  register (username: String, password: String): (user: User)
    **effects** Creates a User with a username and password. 
  authenticate (username: String, password: String): (user: User)
    **requires** a User with username username
    **effects** returns the user with User.username = username if User.password = password. 

1. 

   *usernames must be unique*

2. **Email Extension.** 

**concept** PasswordAuthentication

**purpose** limit access to known users

**principle** after a user registers with a username and a password,
  they can authenticate with that same username and password
  and be treated each time as the same user

**state**
  a set of Users with
      a username String
      a password String
   a set of pendingUsers wiht
      a username String
      a Password String
      a token 

**actions**
  register (username: String, password: String): (user: User)
   **requires** No existing user with username. No pendingUser with username. 
    **effects** Creates a pendingUser with a username and password and generated secret token. 

  confirm(username, token): (user: User)
   **requires** a pending user with pendingUser.username = username.
   **effects** if the token = pendingUser.token create a User with username and password from pendingUser, and delete pending user. 
  authenticate (username: String, password: String): (user: User)
    **requires** a User with username username
    **effects** returns the user with User.username = username if User.password = password. 

---

## Exercise 3: Comparing concepts

**concept** PersonalAccessTokenAuthentication

**purpose**
allow users to generate and use revocable, script-safe credentials to authenticate without using their password

**principle**
after a user creates a personal access token, they can present that token together with their username to authenticate.
Tokens may be created, used, and revoked independently of the user's password.

**state**
a set of Users with
 • username: String
 • password: String (or reference to password auth)
 • tokens: set of Token
a Token has
 • tokenValue: String
 • metadata (name, creation date, optional expiration, scopes)

**actions**
register (username: String, password: String): (user: User)
 effects: same as in PasswordAuthentication
createToken (username: String, password: String, metadata): (token: Token)
 requires: valid username/password
 effects: generate a new token, associate it with user, return token
authenticate (username: String, tokenValue: String): (user: User)
 requires: a token with matching tokenValue exists for user
 effects: returns the user
revokeToken (username: String, tokenValue: String)
 requires: token belongs to user
 effects: remove token from state; authentication with that token will fail

*I might change the documentation to point out that it is a recoverable object that can be used in leu of a password. It is not a credential as much as a backup or recovery number*

---

## Exercise 4: Defining familiar Concepts

Concepts can be designed and understood entirely independently of one another, even if eventually they will be composed with other concepts in a larger application context. Designing and evaluating individual concepts is thus an important skill.

In this exercise, you'll define concepts for some common functionality that you're hopefully familiar with. For each one, you should provide all the standard elements of a concept (name, purpose, operational principle, state and actions), and explain any subtleties of your definition in a brief notes section at the end.

**Pick three of the following:**

### Option A: URL Shortener

**concept** URLShortener

**purpose**
Allow users to create short, easily shareable URLs that redirect to longer target URLs. Supports both user-defined and autogenerated suffixes.

**principle**
After a user creates a short URL (with either a chosen or autogenerated suffix), anyone can use the short URL to be redirected to the original target URL.

**state**
A set of URLMappings with
 suffix: String
 targetURL: String
 creator: User (optional)
 creationDate: DateTime

**actions**
createShortURL(targetURL: String, suffix: String?): (shortURL: String)
 requires: if suffix is provided, no existing URLMapping with that suffix
 effects: creates a new URLMapping with the given targetURL and either the provided suffix or an autogenerated unique suffix; returns the short URL

lookupShortURL(suffix: String): (targetURL: String)
 requires: a URLMapping with the given suffix exists
 effects: returns the targetURL associated with the suffix

deleteShortURL(suffix: String, requester: User)
 requires: requester is the creator of the URLMapping (if creator is set)
 effects: removes the URLMapping; future lookups for this suffix will fail

**notes**
Suffixes must be unique.
If no creator is set, deletion may be restricted to administrators.

---

### Option B: Billable Hours Tracking

**concept** BillableHoursTracker

**purpose**
Enable employees to record and track billable work sessions for projects, supporting accurate client billing.

**principle**
An employee starts a session by selecting a project and describing the work; the session is ended explicitly or automatically if forgotten.

**state**
A set of Projects with
 projectId: String
 client: String
A set of Employees with
 employeeId: String
A set of Sessions with
 sessionId: String
 employee: Employee
 project: Project
 description: String
 startTime: DateTime
 endTime: DateTime? (null if ongoing)
 status: {active, completed, auto-ended}

**actions**
startSession(employee: Employee, project: Project, description: String): (session: Session)
 requires: employee is not already in an active session
 effects: creates a new active Session with current time as startTime

endSession(sessionId: String, endTime: DateTime?): (session: Session)
 requires: session is active and belongs to the employee
 effects: sets endTime (default: now), marks session as completed

autoEndSessions(currentTime: DateTime, maxDuration: Duration)
 effects: for any active session exceeding maxDuration, sets endTime to currentTime, marks as auto-ended

getSessions(employee: Employee, project: Project?): (sessions: Set<Session>)
 effects: returns all sessions for the employee, optionally filtered by project

**notes**
The system may notify employees of long-running sessions.
Auto-ended sessions can be reviewed and edited by employees or managers.

---

### Option C: Conference Room Booking

**concept** ConferenceRoomBooking

**purpose**
Allow users to reserve conference rooms for specific time slots, preventing double-booking.

**principle**
A user can book an available room for a time slot; once booked, the room is unavailable for overlapping bookings.

**state**
A set of Rooms with
 roomId: String
 name: String
A set of Bookings with
 bookingId: String
 room: Room
 user: User
 startTime: DateTime
 endTime: DateTime

**actions**
bookRoom(user: User, room: Room, startTime: DateTime, endTime: DateTime): (booking: Booking)
 requires: room is available (no overlapping Booking) for the requested time slot
 effects: creates a new Booking for the room and time slot

cancelBooking(bookingId: String, requester: User)
 requires: requester is the user who made the booking or an admin
 effects: removes the Booking

getBookings(room: Room, date: Date): (bookings: Set<Booking>)
 effects: returns all Bookings for the room on the given date

**notes**
- Overlapping bookings are not allowed.
- Additional policies (e.g., advance notice, booking limits) can be layered on top of this concept.
