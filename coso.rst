

Tutorial
========

See the tutorial `here <_docs/tutorial.md>`__


Forms
=====

Each page in oTree can contain a form, which the player should fill out
and submit by clicking the "Next" button. To create a form, first you
should go to models.py and define fields on your Player or Group. Then,
in your Page class, you can define ``form_models`` to specify he model
that this form modifies (either ``models.Player`` or ``models.Group``),
and ``form_fields``, which is list of the fields from that model.

When the user submits the form, the submitted data is automatically
saved back to the field in your model.

Forms in templates
------------------

You should include form fields by using a ``{% formfield %}`` element.
You generally do not need to write raw HTML for forms (e.g.
``<input type="text" id="...">``).

User Input Validation
---------------------

The player must submit a valid form before they get routed to the next
page. If the form they submit is invalid (e.g. missing or incorrect
values), it will be re-displayed to them along with the list of errors
they need to correct.

*Example 1:* |image0|

*Example 2:* |image1|

oTree automatically validates all input submitted by the user.t For
example, if you have a form containing a ``PositiveIntegerField``, oTree
will not let the user submit values that are not positive integers, like
``-1``, ``1.5``, or ``hello``.

Additionally, you can customize validation by passing extra arguments to
your model field definition. For example, if you want to require a
number to be between 12 and 24, you can specify it like this:

.. code:: python

    offer = models.PositiveIntegerField(min=12, max=24)

If you specify a ``choices`` argument, the default form widget will be a
select box with these choices instead of the standard text field.

.. code:: python

    year_in_school = models.CharField(choices=['Freshman', 'Sophomore', 'Junior', 'Senior'])

If you would like a specially formatted value displayed to the user that
is different from the values stored internally, you can return a list
consisting itself of tuples of two items to use as choices for this
field. The first element in each tuple is the actual value to be set on
the model, and the second element is the human-readable name. For
example:

.. code:: python

    year_in_school = models.CharField(
        choices=[
            ('FR', 'Freshman'),
            ('SO', 'Sophomore'),
            ('JR', 'Junior'),
            ('SR', 'Senior'),
        ]
    )

After the field has been set, you can access the human-readable name
using
`get\_FOO\_display <https://docs.djangoproject.com/en/1.8/ref/models/instances/#django.db.models.Model.get_FOO_display>`__
method, like this:
``self.get_year_in_school_display() # returns e.g. 'Sophomore'``

If a field is optional, you can do:

.. code:: python

    offer = models.PositiveIntegerField(blank=True)

Dynamic validation
~~~~~~~~~~~~~~~~~~

If you need a form's choices or validation logic to depend on some
dynamic calculation, then you can instead define one of the below
methods in your ``Page`` class in ``views.py``.

-  ``def {field_name}_choices(self)``

Example:

.. code:: python

    def offer_choices(self):
        return currency_range(0, self.player.endowment, 1)

-  ``def {field_name}_min(self)``

The dynamic alternative to ``min``.

-  ``def {field_name}_max(self)``

The dynamic alternative to ``max``.

-  ``def {field_name}_error_message(self, value)``

This is the most flexible method for validating a field.

For example, let's say your form has an integer field called
``odd_negative``, which must be odd and negative: You would enforce this
as follows:

.. code:: python

    def odd_negative_error_message(self, value):
        if not (value < 0 and value % 2):
            return 'Must be odd and negative'

Validating multiple fields together
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's say you have 3 integer fields in your form whose names are
``int1``, ``int2``, and ``int3``, and the values submitted must sum to
100. You would define the ``error_message`` method in your Page class:

.. code:: python

    def error_message(self, values):
        if values["int1"] + values["int2"] + values["int3"] != 100:
            return 'The numbers must add up to 100'

Determining the list of form fields dynamically
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you need the list of form fields to be dynamic, instead of
``form_fields`` you can define a method ``get_form_fields(self)`` that
returns the list.

Widgets
-------

The full list of form input widgets offered by Django is
`here <https://docs.djangoproject.com/en/1.7/ref/forms/widgets/#built-in-widgets>`__.

oTree additionally offers ``RadioSelectHorizontal`` and ``SliderInput``.

Custom widgets and hidden fields
--------------------------------

It's not mandatory to use the ``{% formfield %}`` element; you can write
the raw HTML for any form input if you wish to customize its behavior or
appearance. Just include an ``<input>`` element with the same ``name``
attribute as the field. For example, if you want a hidden input, you can
do this:

::

    # models.py
    my_hidden_input = models.PositiveIntegerField()

    # views.py
    form_fields = ['my_hidden_input', 'some_other_field']

    # HTML template
    <input type="hidden" name="my_hidden_input" value="5" id="id_my_hidden_input"/>

Then you can use JavaScript to set the value of that input, by selecting
the element by id "id\_my\_hidden\_input".

For simple widgets you can use jQuery; for more complex or custom form
interfaces, you can use a front-end framework with databinding, like
React or Polymer.

If you want your custom widget's style to look like the rest of the
oTree widgets, you should look at the generated HTML from the {%
formfield %} tag. You can copy and paste the markup into the template
and use that as a starting point for modifications.

Object model and ``self``
=========================

In oTree code, you will see the variable ``self`` all throughout the
code. In Python, ``self`` refers to the object whose class you are
currently in.

For example, in this example, ``self`` refers to a ``Player`` object:

::

    class Player(object):

        def my_method(self):
            return self.my_field

In the next example, however, ``self`` refers to a ``Group`` object:

::

    class Group(object):

        def my_method(self):
            return self.my_field

``self`` is conceptually similar to the word "me". You refer to yourself
as "me", but others refer to you by your name. And when your friend says
the word "me", it has a different meaning from when you say the word
"me".

oTree's different objects are all connected, as illustrated by this
diagram.

.. figure:: https://i.imgur.com/foFSxix.jpg
   :alt:

Player, group, subsession, and session are in a simple hierarchy,
'session' being at the top and 'player' being at the bottom. Then, the
'page' has an arrow to all 4 of these objects.

For example, if you are in a method on the ``Player`` class, you can
access the player's payoff with ``self.payoff`` (because ``self`` is the
player). But if you are inside a ``Page`` class in ``views.py``, the
equivalent expression you instead need to do ``self.player.payoff``,
which traverses the pointer from 'page' to 'player'.

Here are some code examples to illustrate:

.. code:: python

    class Session(...) # this class is defined in oTree-core
        def example(self):

            # current session object
            self

            # parent objects
            self.session_type

            # child objects
            self.get_subsessions()
            self.get_participants()

    class Participant(...) # this class is defined in oTree-core
        def example(self):

            # current participant object
            self

            # parent objects
            self.session

            # child objects
            self.get_players()

    # in your models.py
    class Subsession(otree.models.Subsession):
        def example(self):

            # current subsession object
            self

            # parent objects
            self.session

            # child objects
            self.get_groups()
            self.get_players()

            # accessing previous Subsession objects
            self.in_previous_rounds()
            self.in_all_rounds()

    class Group(otree.models.Group):
        def example(self):

            # current group object
            self

            # parent objects
            self.session
            self.subsession

            # child objects
            self.get_players()

    class Player(otree.models.Player):

        def example(self):

            # current player object
            self

            # method you defined on the current object
            self.my_custom_method()

            # parent objects
            self.session
            self.subsession
            self.group
            self.participant

            self.session.session_type

            # accessing previous player objects
            self.in_previous_rounds()
            self.in_all_rounds() # equivalent to self.in_previous_rounds() + [self]

    # in your views.py
    class MyPage(Page):
        def example(self):

            # current page object
            self

            # parent objects
            self.session
            self.subsession
            self.group
            self.player

            # example of chaining lookups
            self.player.participant
            self.session.session_type

You can follow pointers in a transitive manner. For example, if you are
in the Page class, you can access the participant as
``self.player.participant``. If you are in the Player class, you can
access the session type as ``self.session.session_type``.

Groups and multiplayer games
============================

In oTree, you can define multiplayer interactive games through the use
of groups

To do this, go to your app's models.py and set
``Constants.players_per_group``. For example, in a 2-player game like an
ultimatum game or prisoner's dilemma, you would set this to 2. If your
app does not involve dividing the players into multiple groups, then set
it to ``None``. e.g. it is a single-player game or a game where
everybody in the subsession interacts together as 1 group. In this case,
``self.group.get_players()`` will return everybody in the subsession. If
you need your groups to have uneven sizes (for example, 2 vs 3), you can
do this: ``players_per_group=[2,3]``; in this case, if you have a
session with 15 players, the group sizes would be ``[2,3,2,3,2,3]``.

Each player has a numeric field ``id_in_group``. This is useful in
multiplayer games where players have different roles, so that you can
determine if the player is player 1, player 2, or so on.

Groups have the following methods:

-  ``get_players()``: returns a list of the players in the group.
-  ``get_player_by_id(n)``: Retrieves the player in the group with a
   specific ``id_in_group``.
-  ``get_player_by_role(r)``. The argument to this method is a string
   that looks up the player by their role value. (If you use this
   method, you must define the ``role`` method on the player model,
   which should return a string that depends on ``id_in_group``.)

Player objects have methods ``get_others_in_group()`` and
``get_others_in_subsession()`` that return a list of the other players
in the group and subsession, respectively.

Wait pages
----------

Wait pages are necessary when one or more players need to wait for
another player to take some action before they can proceed. For example,
in an ultimatum game, player 2 cannot accept or reject before they have
seen player 1's offer.

Wait pages are defined in views.py. If you have a wait page in your
sequence of pages, then oTree waits until all players in the group have
arrived at that point in the sequence, and then all players are allowed
to proceed.

If your subsession has multiple groups playing simultaneously, and you
would like a wait page that waits for all groups (i.e. all players in
the subsession), you can set the attribute
``wait_for_all_groups = True`` on the wait page.

Wait pages can define the following methods:

-  ``def after_all_players_arrive(self)``

This code will be executed once all players have arrived at the wait
page. For example, this method can determine the winner of an auction
and set each player's payoff.

-  ``def title_text(self)``

The text in the title of the wait page.

-  ``def body_text(self)``

The text in the body of the wait page

Group re-matching between rounds
--------------------------------

For the first round, the players are split into groups of
``Constants.players_per_group``. This matching is random, unless you
have set ``group_by_arrival_time`` set in your session type in
settings.py, in which case players are grouped in the order they start
the first round.

In subsequent rounds, by default, the groups chosen are kept the same.
If you would like to change this, you can define the grouping logic in
``Subsession.before_session_starts``. For example, if you want players
to be reassigned to the same groups but to have roles randomly shuffled
around within their groups (e.g. so player 1 will either become player 2
or remain player 1), you would do this:

.. code:: python

    def before_session_starts(self):
        for group in self.get_groups():
            players = group.get_players()
            players.reverse()
            group.set_players(players)

A group has a method ``set_players`` that takes as an argument a list of
the players to assign to that group, in order. Alternatively, a
subsession has a method ``set_groups`` that takes as an argument a list
of lists, with each sublist representing a group. You can use this to
rearrange groups between rounds, but note that the
``before_session_starts`` method is run when the session is created,
before players begin playing. Therefore you cannot use this method to
shuffle players depending on the results of previous rounds (there is a
separate technique for doing this which will be added to the
documentation in the future).

Money and Points
================

In many experiments, participants play for currency: either virtual
points, or real money. oTree supports both scenarios. Participants can
be awarded a fixed base pay (i.e. participation fee). In addition, in
each subsession, they can be awarded an additional payoff.

You can specify the payment currency in ``settings.py``, by setting
``REAL_WORLD_CURRENCY_CODE`` to "USD", "EUR", "GBP", and so on. This
means that all currency amounts the participants see will be
automatically formatted in that currency, and at the end of the session
when you print out the payments page, amounts will be displayed in that
currency.

In oTree apps, currency values have their own data type. You can define
a currency value with the ``c()`` function, e.g. ``c(10)`` or ``c(0)``.
Correspondingly, there is a special model field for currency values:
``CurrencyField``. For example, each player has a ``payoff`` field,
which is a ``CurrencyField``. Currency values work just like numbers
(you can do mathematical operations like addition, multiplication, etc),
but when you pass them to an HTML template, they are automatically
formatted as currency. For example, if you set
``player.payoff = c(1.20)``, and then pass it to a template, it will be
formatted as ``$1.20`` or ``1,20 €``, etc., depending on your
``REAL_WORLD_CURRENCY_CODE`` and ``LANGUAGE_CODE`` settings.

Note: instead of using Python's built-in ``range`` function, you should
use oTree's ``currency_range`` with currency values. It takes 3
arguments (start, stop, step), just like range. However, note that it is
an inclusive range. For example,
``currency_range(c(0), c(0.10), c(0.02))`` returns something like:

.. code:: python

    [Money($0.00), Money($0.02), Money($0.04), Money($0.06), Money($0.08), Money($0.10)]

Assigning payoffs
-----------------

Each player has a ``payoff`` field, which is a ``CurrencyField``. If
your player makes money, you should store it in this field.
``player.participant.payoff`` is the sum of the payoffs a participant
made in each subsession. At the end of the experiment, a participant's
total profit is calculated by adding the fixed base pay to the
``payoff`` that participant earned as a player in each subsession.

Points (i.e. "experimental currency")
-------------------------------------

Sometimes it is preferable for players to play games for points or
"experimental currency units", which are converted to real money at the
end of the session. You can set ``USE_POINTS = True`` in
``settings.py``, and then in-game currency amounts will be expressed in
points rather than real money.

For example, ``c(10)`` is displayed as ``10 points``. You can specify
the conversion rate to real money in ``settings.py`` by providing a
``real_world_currency_per_point`` key in the session type dictionary.
For example, if you pay the user 2 cents per point, you would set
``real_world_currency_per_point = 0.02``.

You can convert a point amount to money using the
``to_real_world_currency()`` method, which takes as an argument the
current session (this is necessary because different sessions can have
different conversion rates).

Let's say ``real_world_currency_per_point = 0.02``

.. code:: python

    c(10) # evaluates to Currency(10 points)
    c(10).to_real_world_currency(self.session) # evaluates to $0.20

Treatments
==========

If you want to assign participants to different treatment groups, you
can put the code in the subsession's ``before_session_starts`` method.
For example, if you want some participants to have a blue background to
their screen and some to have a red background, you would randomize as
follows:

.. code:: python

    def before_session_starts(self):
        # randomize to treatments
        for player in self.get_players():
            player.color = random.choice(['blue', 'red'])

(To actually set the screen color you would need to pass
``player.color`` to some CSS code in the template, but that part is
omitted here.)

If your game has multiple rounds, note that the above code gets executed
for each round. So if you want to ensure that participants are assigned
to the same treatment group each round, you should set the property at
the participant level, which persists across subsessions, and only set
it in the first round:

.. code:: python

    def before_session_starts(self):
        if self.round_number == 1:
            for p in self.get_players():
                p.participant.vars['color'] = random.choice(['blue', 'red'])

Then, in the view code, you can access the participant's color with
``self.player.participant.vars['color']``.

Choosing which treatment to play
--------------------------------

In the above example, players got randomized to treatments. This is
useful in a live experiment, but when you are testing your game, it is
often useful to choose explicitly which treatment to play. Let's say you
are developing the game from the above example and want to show your
colleagues both treatments (red and blue). You can create 2 session
types in settings.py that have the same keys to session type dictionary
, except the ``treatment`` key:

.. code:: python

    SESSION_TYPES = [
        {
            'name':'my_game_blue',
            # other arguments...

            'treatment':'blue',

        },
        {
            'name':'my_game_red',
            # other arguments...
            'treatment':'red',
        },
    ]

Then in the ``before_session_starts`` method, you can check which of the
2 session types it is:

.. code:: python

    def before_session_starts(self):
        for p in self.get_players():
            if 'treatment' in self.session.session_type:
                # demo mode
                p.color = self.session.session_type['treatment']
            else:
                # live experiment mode
                p.color = random.choice(['blue', 'red'])

Then, when someone visits your demo page, they will see the "red" and
"blue" treatment, and choose to play one or the other. If the demo
argument is not passed, the color is randomized.

Rounds
======

In oTree, "rounds" and "subsessions" are almost synonymous. The
difference is that "rounds" refers to a sequence of subsessions that are
in the same app. So, a session that consists of a prisoner's dilemma
iterated 3 times, followed by an exit questionnaire, has 4 subsessions,
which consists of 3 rounds of the prisoner's dilemma, and 1 round of the
questionnaire.

Round numbers
-------------

You can specify how many rounds a game should be played in models.py, in
``Constants.num_rounds``.

Subsession objects have an attribute ``round_number``, which contains
the current round number, starting from 1.

Accessing data from previous rounds
-----------------------------------

Player objects have methods ``in_previous_rounds()`` and
``in_all_rounds()`` that each return a list of players representing the
same participant in previous rounds of the same app. The difference is
that ``in_all_rounds()`` includes the current round's player. For
example, if you wanted to calculate a participant's payoff for all
previous rounds of a game, plus the current one:

.. code:: python

    cumulative_payoff = sum([p.payoff for p in self.player.in_all_rounds()])

Similarly, subsession objects have methods ``in_previous_rounds()`` and
``in_all_rounds()`` that work the same way.

Accessing data from previous subsessions in different apps
----------------------------------------------------------

``in_all_rounds()`` only is useful when you need to access data from a
previous round of the same app. If you want to pass data between
subsessions of different app types (e.g. the participant is in the
questionnaire and needs to see data from their ultimatum game results),
you should store this data in the participant object, which persists
across subsessions. Each participant has a field called ``vars``, which
is a dictionary that can store any data about the player. For example,
if you ask the participant's name in one subsession and you need to
access it later, you would store it like this:

``self.player.participant.vars['first name'] = 'Chris'``

Then in a future subsession, you would retrieve this value like this:

``self.player.participant.vars['first name']`` # returns 'Chris'

Global variables
----------------

For session-wide globals, you can use ``self.session.vars``.

This is a dictionary just like ``participant.vars``.

Trying your app
===============

You can launch your app on your local development machine to test it,
and then when you are satisfied, you can deploy it to a server.

Testing locally
~~~~~~~~~~~~~~~

You will be testing your app frequently during development, so that you
can see how the app looks and feels and discover bugs during
development. To test your app, run the server in the oTree launcher. You
may need to reset the database first.

Click on a session name and you will get a start link for the
experimenter, as well as the links for all the participants. You can
open all the start links in different tabs and simulate playing as
multiple participants simultaneously.

You can send the demo page link to your colleagues or publish it to a
public audience.

Debugging
~~~~~~~~~

Once you start playing your app, you will sometimes get a yellow Django
error page with lots of details. To troubleshoot this, look at the error
message and "Exception location" fields. If the exception location is
somewhere outside of your app's code (like if it points to an installed
module like Django or oTree), look through the "Traceback" section to
see if it passes through your code. Once you have found a reference to a
line of code in your app, go to that line of code and see if the error
message can help you pinpoint an error in your code. Googling the error
name or message will often take you to pages that explain the meaning of
the error and how to fix it.

Debugging with PyCharm
^^^^^^^^^^^^^^^^^^^^^^

PyCharm has an excellent debugger that you should be using continuously.
You can insert a breakpoint into your code by clicking in the left-hand
margin on a line of code. You will see a little red dot. Then reload the
page and the debugger will pause when it hits your line of code. At this
point you can inspect the state of all the local variables, execute
print statements in the built-in intepreter, and step through the code
line by line.

More on the PyCharm debugger
`here <http://www.jetbrains.com/pycharm/webhelp/debugging.html>`__.

Test Bots
=========

Automated tests are an essential part of building a oTree app. You can
easily program a bot that simulates multiple players simultaneously
playing your app.

Tests with dozens of bots complete with in seconds, and afterward
automated tests can be run to verify correctness of the app (e.g. to
ensure that payoffs are being calculated correctly).

This automated test system saves the programmer the effort of having to
re-test the application every time something is changed.

Launching tests
~~~~~~~~~~~~~~~

oTree tests entire sessions, rather that individual apps in isolation.
This is to make sure the entire session runs, just as participants will
play it in the lab.

Let's say you want to test the session named ``ultimatum`` in
``settings.py``. To test, click the "Terminal" button in the oTree
launcher run the following command from your project's root directory:

.. code:: bash

    python otree test ultimatum_game

This command will test the session, with the number of participants
specified in ``settings.py``. For example, ``num_bots`` is 30, then when
you run the tests, 30 bots will be instantiated and will play
concurrently.

To run tests for all sessions in ``settings.py``, run:

.. code:: bash

    python otree test

Writing tests
~~~~~~~~~~~~~

Tests are contained in your app's ``tests.py``. Fill out the
``play_round()`` method of your ``PlayerBot`` (and ``ExperimenterBot``
if you have experimenter pages). It should simulate each page
submission. For example:

.. code:: python

    self.submit(views.Start)
    self.submit(views.Offer, {'offer_amount': 50})

Here, we first submit the ``Start`` page, which does not contain a form.
The next page is ``Offer``, which contains a form whose field is called
``offer_amount``, which we set to ``50``. This is a way of automating
the task of

Your test bot must simulate playing the game correctly. The bot in the
above example would raise an error if the page after ``Start`` was
called ``Instructions`` rather than ``Offer``, or if the field
``offer_amount`` was actually called something else. Your test bot is a
specification of how you expect your app to work, so when it raises an
error, it will alert you that your app is not behaving as intended.

Rather than programming many separate bots, you program one bot that can
play any variation of the game. For example, if you have different
treatment groups that play different pages, you can branch by checking a
variable on the treatment. For example, here is how you would play if
one treatment group sees a "threshold" page but the other treatment
group should see an "accept" page:

.. code:: python

    if self.group.threshold:
        self.submit(views.Threshold, {'offer_accept_threshold': 30})
    else:
        self.submit(views.Accept, {'offer_accepted': True})

If player 1 in a group sees different pages from player 2, you can
define separate methods ``play_p1()`` and ``play_p2()`` and branch like
this:

.. code:: python

    if self.player.id_in_group == 0:
        self.play_p1()
    else:
        self.play_p2()

To get the maximal benefit, your bot should thoroughly test all parts of
your code. Here are some ways you can test your app:

-  Ensure that it correctly rejects invalid input. For example, if you
   ask the user to enter a number that is a multiple of 3, you can
   verify that entering 4 will be rejected by using the
   ``submit_invalid`` method as follows. This line of code will raise an
   error if the submission is *accepted*:

   ``self.submit_invalid(views.EnterNumber, {'multiple_of_3': 4})``

-  You can put assert statements in the bot's ``validate_play()`` method
   to check that the correct values are being stored in the database.
   For example, if a player's bonus is defined to be 100 minus their
   offer, you can check your program is calculating it correctly as
   follows:

   ``self.submit(views.Offer, {'offer': 30})``

   ``assert self.player.bonus == 70``

-  You can use random amounts to test that your program can handle any
   type of random input:

   ``self.submit(views.Offer, {'offer': random.randint(0,100)})``

Bots can either be programmed to simulate playing the game according to
an ordinary strategy, or to test "boundary conditions" (e.g. by entering
invalid input to see if the application correctly rejects it). Or yet
the bot can enter random input on each page.

If your app has [[Experimenter Pages]], you can also implement the
``play`` method on the ``ExperimenterBot``.

Admin
=====

oTree comes with an admin interface, so that experimenters can manage
sessions, monitor the progress of live sessions, and export data after
sessions.

Open your browser to the root url of your web application. If you're
developing locally, this will be http://127.0.0.1:8000/.

Lab Experiments
---------------

Creating sessions
~~~~~~~~~~~~~~~~~

Create a session in the admin. [TODO: more info]

Opening links
~~~~~~~~~~~~~

To launch a session, each participant must open their link. There are 2
options for how to open URLs.

Lab
^^^

In the admin interface, go to the "global data" section, and copy the
"lab link". This is a permanent URL that will last as long as you use
the same server [TODO: finish]

Each workstation has a permanent URL that, when clicked, will route the
participant to the currently active session.

choose an active session from the dropdown. Then, copy

Unique URLs
^^^^^^^^^^^

If you are running your experiment in a lab, you should deploy the links
to the target workstations using whatever means is available to you. If
you have a tool that can push distinct URLs to each PC in the lab, you
can use that. Or you can set up a unique email account for each
workstation, and then send the unique links to PCs using a mail merge.
Then open the link on each PC.

Participant labels
^^^^^^^^^^^^^^^^^^

oTree uses a unique code to identify each participant. However, you can
assign each participant a "label" that can be any convenient way to
identify them to you, such as:

-  Name
-  Computer workstation number
-  Email address
-  ID number

This label will be displayed in places where participants are listed,
like the oTree admin interface or the payments page.

You can assign each participant a label by adding a parameter to each
start link. For example, if you want to assign a participant the label
"WORKSTATION\_1", you would take the start link for that participant:

::

    http://[participant's start link]

And change it to:

::

    http://[participant's start link]?participant_label=WORKSTATION_1

Outside of oTree, you can create a script that adds a unique
``participant_label`` to each start link as indicated above. Then, when
the link is opened, the oTree server will register that participant
label for that participant.

Monitor sessions
~~~~~~~~~~~~~~~~

While your session is ongoing, you can monitor the live progress in the
admin interface. The admin tables update live, highlighting changes as
they occur. The most useful table to monitor is "Session participants",
which gives you a summary of all the participants' progress. You can
also click the "participants" table of each app to see the details of
all the data being entered in your subsessions.

Authenticaton
-------------

When you first install oTree, The entire admin interface is accessible
without a password. However, when you are ready to launch your oTree
app, you should password protect the admin so that visitors and
participants cannot access sensitive data.

If you are launching an experiment and want visitors to only be able to
play your app if you provided them with a start link, set the
environment variable ``OTREE_AUTH_LEVEL`` to ``EXPERIMENT``.

If you would like to put your site online in public demo mode where
anybody can play a demo version of your game, set ``OTREE_AUTH_LEVEL``
to ``DEMO``. This will allow people to play in demo mode, but not access
the full admin interface.

Online experiments
------------------

Experiments can be launched to participants playing over the internet,
in a similar way to how experiments are launched the lab. Login to the
admin, create a session, then distribute the links to participants via
email or a website.

In a lab, you usually can start all participants at the same time, but
this is often not possible online, because some participants might click
your link hours after other participants. If your game requires players
to play in groups, you may want to set the ``group_by_arrival_time`` key
in session type dictionary to ``True``. This will group players in the
order in which they arrive to your site, rather than randomly, so that
players who arrive around the same time play with each other.

Kiosk Mode
----------

During an experiment, subjects are expected to stay on the given
pages/game, instead of browsing irrelevant websites or using other
applications. Kiosk mode locks down the oTree pages on a web browser
thus allows the subjects to focus. Here we provide some guidelines to
initiate Kiosk mode with different browsers/on various systems. In
general, Kiosk mode is rather user-friendly so one can easily search
online how to use it on specific platforms.

iOS (iPhone/iPad)
^^^^^^^^^^^^^^^^^

1. Go to Setting – Accessibility – Guided Access
2. Turn on Guided Access and set a passcode for your Kiosk mode
3. Open your web browser and enter your URL
4. Triple-click home button to initiate Kiosk mode
5. Circle areas on the screen to disable (e.g. URL bar) and activate

Android
^^^^^^^

There are several apps for using Kiosk mode on Android, for instance:
`Kiosk Browser
Lockdown <https://play.google.com/store/apps/details?id=com.procoit.kioskbrowser&hl=en>`__.

.. figure:: http://i.imgur.com/VJ72fKv.png
   :alt:

For iOS and Android tablets, Kiosk mode will continue to function after
normal restart. However, if subjects enter Android safe mode, the app
can be disabled.

Chrome on PC
^^^^^^^^^^^^

1. Go to Setting – Users – Add new user
2. Create a new user with a desktop shortcut
3. Right-click the shortcut and select “Properties”
4. In the “Target” filed, add to the end either
   ``--kiosk "http://www.your-otree-server.com"`` or
   ``--chrome-frame  --kiosk "http://www.your-otree-server.com"``
5. Disable hotkeys (see
   `here <http://superuser.com/questions/727072/what-windows-shortcuts-should-be-blocked-on-a-kiosk-mode-pc>`__)
6. Open the shortcut to activate Kiosk mode

IE on PC
^^^^^^^^

IE on PC See `here <http://support2.microsoft.com/kb/154780>`__

Mac
^^^

There are several apps for using Kiosk mode on Mac, for instance:
`eCrisper <http://ecrisper.com/>`__. Mac keyboard shortcuts should be
disabled.

Payment PDF
-----------

At the end of your session, you can open and print a page that lists all
the participants and how much they should be paid.

.. figure:: http://i.imgur.com/nSMlWcY.png
   :alt:

Export Data
-----------

You can download your raw data in text format (CSV) so that you can view
and analyze it with a program like Excel, Stata, or R.

Autogenerated documentation
---------------------------

Each model field you define can also have a ``doc=`` argument. Any
string you add here will be included in the autogenerated documentation
file, which can be downloaded through the data export page in the admin.

Debug Info
----------

Any application can be run so that that debug information is displayed
on the bottom of all screens. The debug information consists of the ID
in group, the group, the player, the participant label, and the session
code. The session code and participant label are two randomly generated
alphanumeric codes uniquely identifying the session and participant. The
ID in group identifes the role of the player (e.g., in a principal-agent
game, principals might have the ID in group 1, while agents have 2).

.. figure:: http://i.imgur.com/DZsyhQf.png
   :alt:

Progress-Monitor
----------------

The progress monitor allows the researcher to monitor the progress of an
experiment. It features a display that can be **filtered** and
**sorted**, for example by computer name or group. The experimenter can
see the progress of all participants, including their current action and
taken decisions. Updates are shown as they happen **in real time** and
cells that change are highlighted in yellow. Because the progress
monitor is web-based, **multiple collaborators can simultaneously open
it on several devices on premises or at remote locations**.

.. figure:: http://i.imgur.com/0nYKnDp.png
   :alt:

Session Interface
-----------------

The session interface is an optional feature convenient in some
experiments. In many experimental settings, in addition to monitoring,
**an experimenter needs to receive instructions or provide input for the
experiment**. The session interface can instruct an experimenter on what
to do next and show text to be read aloud. The session interface can
also request input from the experimenter at a specic point in the
session. For example, in an Ellsberg experiment, the experimenter might
roll an opaque urn prior to the session; the session interface will
remind the experimenter to show the urn to the participants, tell the
experimenter when all participants have selected their bets, and
instruct her to draw a ball from the urn. It will then ask the drawn
color, so that oTree can calculate participants' payoffs.

Deploying to a server
=====================

oTree can be deployed on your own server, or using a cloud service like
Heroku.

If you are not experienced with web server administration, Heroku may be
a much simpler option for you, because Heroku automatically handles much
of the configuration. Instructions on how to deploy oTree to Heroku are
`here <#heroku>`__.

Nevertheless, in various situations it will be preferable to run oTree
on your own server. Reasons may include:

-  You do not want your server to be accessed from the internet
-  You will be launching your experiment in a setting where internet
   access is unavailable
-  You want full control over how your server is configured

oTree runs on top of Django, so oTree setup is the same as Django setup.
Django runs on a wide variety of servers, except getting it to run on
Windows may require extra work.

The most typical setup will be a Linux server with Apache. The
instructions for this setup are
`here <https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/modwsgi/>`__.

If you have been developing your project on your local PC, you should
push your oTree folder to your webserver, e.g. with Git. Then, you
should make sure your webserver has Python installed (possibly in a
``virtualenv``), and do ``pip install -r requirements.txt`` to install
all the dependencies. When you are ready to launch the experiment, you
should set ``OTREE_PRODUCTION`` to ``1``, to turn off ``DEBUG`` mode.

Heroku
------

Here are the steps for deploying to Heroku.

Create an account
~~~~~~~~~~~~~~~~~

Create a free account on `Heroku <https://www.heroku.com/>`__. You can
skip the "Getting Started With Python" guide.

Install the Heroku Toolbelt
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install the `Heroku Toolbelt <https://toolbelt.heroku.com/>`__.

This provides you access to the Heroku Command Line utility.

Once installed, you can use the ``heroku`` command from your command
shell.

From the oTree launcher, click the terminal button to access the command
shell. Log in using the email address and password you used when
creating your Heroku account:

::

    $ heroku login
    Enter your Heroku credentials.
    Email: python@example.com
    Password:
    Authentication successful.
    Authenticating is required to allow both the heroku and git commands to operate.

Create the Heroku app
~~~~~~~~~~~~~~~~~~~~~

Create an app on Heroku, which prepares Heroku to receive your source
code:

::

    $ heroku create
    Creating lit-bastion-5032 in organization heroku... done, stack is cedar-14
    http://lit-bastion-5032.herokuapp.com/ | https://git.heroku.com/lit-bastion-5032.git
    Git remote heroku added
    When you create an app, a git remote (called heroku) is also created and associated with your local git repository.

Heroku generates a random name (in this case lit-bastion-5032) for your
app, or you can pass a parameter to specify your own app name.

Deploy your code
~~~~~~~~~~~~~~~~

``cd`` to the root directory of your oTree project.

Make sure you have committed any changes as follows:

::

    $ git add .
    $ git commit -am '[commit message]'

(If you get the message
``fatal: Not a git repository (or any of the parent directories): .git``
then you first need to initialize the git repo.)

Then do:

::

    $ git push heroku master
    $ python otree-heroku resetdb myherokuapp

Now visit the app at the URL generated by its app name. As a handy
shortcut, you can open the website as follows:

::

    $ heroku open

To set environment variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If it's a production website, you should set the environment variables
(e.g. ``OTREE_PRODUCTION`` and ``OTREE_AUTH_LEVEL``), like this:

.. code:: bash

    heroku config:set OTREE_PRODUCTION=1
    heroku config:set OTREE_AUTH_LEVEL=DEMO

To add an existing remote:
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    heroku git:remote -a [myherokuapp]

Database setup
--------------

oTree is most frequently used with PostgreSQL as the production
database. You can create your database with a command like this:

``psql -c 'create database django_db;' -U postgres``

Then, you should set the following environment variable, so that it can
be read by ``dj_database_url``:

``DATABASE_URL=postgres://postgres@localhost/django_db``

Amazon Mechanical Turk
======================

Overview
--------

oTree provides integration with Amazon Mechanical Turk (AMT).

You can publish your game to Amazon mechanical Turk directly from
oTree's admin interface. Then, workers on mechanical Turk can accept and
play your app as an MTurk HIT and get paid a participation fee as well
as bonuses they earned by playing your game.

Server requirements
-------------------

Amazon MTurk requires the support of SSL by the server on which you
deploy oTree. That is you should be able to access your server with the
following link ``https://www.myserver.com``.

There are two Out of the Box solutions that oTree supports:

-  running your experiment on oTree local SSL server with the following
   command ``python otree runsslserver``. This SSL server is fully
   compatible with oTree and can be used interchangeably with regular
   server ``python otree runserver``.
-  deploying oTree on Heroku. Heroku by default support SSL. Deploying
   to heroku is explained `here <#heroku>`__.

AWS credentials
---------------

Researchers must have an employer account with AMT, which currently
requires a U.S. address and bank account.

To make payments to participants you need to generate
``AWS_ACCESS_KEY_ID`` and ``AWS_SECRET_ACCESS_KEY``
`here <https://console.aws.amazon.com/iam/home?#security_credential>`__:

.. figure:: http://i.imgur.com/dNhkOiA.png
   :alt: AWS key

   AWS key
On heroku add generated values to your environment variables:

.. code:: bash

    heroku config:set AWS_ACCESS_KEY_ID=YOUR_AWS_ACCESS_KEY_ID
    heroku config:set AWS_SECRET_ACCESS_KEY=YOUR_AWS_SECRET_ACCESS_KEY

Making your session work on MTurk
---------------------------------

You should look in ``settings.py`` for all settings related to
Mechanical Turk (do a search for "mturk"). You can edit the properties
of the HIT such as the title and keywords, as well as the qualifications
required to participate. The monetary reward paid to workers is the
``participation_fee`` for your ``session_type``.

When you publish your HIT to MTurk, it will be visible to workers. When
a worker clicks on the link to take part in the HIT, they will see the
MTurk interface, with your app loaded inside a frame (as an
``ExternalQuestion``). Initially, they will be in preview mode, and will
see the ``preview_template`` you specify in ``settings.py``. After they
accept the HIT, they will see the first page of your session, and be
able to play your session while it is embedded inside a frame in the
MTurk worker interface.

The only modification you should make to your app for it to work on AMT
is to add a ``{% next_button %}`` to the final page that your
participants see. When the participant clicks this button, they will be
directed back to the mechanical Turk website and their work will be
submitted.

After workers have completed the session, you can click on the
"payments" Tab for your session. Here, you will be able to approve
submissions, and also pay the bonuses that workers earned in your game.

Testing your hit in sandbox
---------------------------

The Mechanical Turk Developer Sandbox is a simulated environment that
lets you test Human Intelligence Tasks (HITs) prior to publication in
the marketplace. This environment is available for both
`worker <https://workersandbox.mturk.com/mturk/welcome>`__ and
`requester <https://requester.mturk.com/developer/sandbox>`__.

After publishing the HIT you can test it both as a worker and as a
requester using the links provided on "MTurk" Tab of your session admin
panel. These links will work only locally given that you created your
HIT being on local server(\ ``python otree runsslserver``).

Multiplayer games
-----------------

Games that involve synchronous interaction between participants (i.e.
wait pages) can be tricky on mechanical Turk. First, you should set
``group_by_arrival_time`` as ``True`` so that participants are assigned
to groups in the order in which they arrive, to minimize unnecessary
waiting time.

However, there is still the issue that if one participant drops out then
other participants might be stuck on a wait page. One way to mitigate
this attrition problem is to use a "lock-in" task. In other words,
before your multiplayer game, you can have a subsession that is a
single-player app and takes some effort to complete. The idea is that a
participant takes the effort to complete this initial task, they are
less likely to drop out after that point. Then, the first few
participants to finish the lock in task will be assigned to the same
group in the next subsession, which is the multiplayer game.

An upcoming feature in oTree that has not yet been implemented is the
ability to auto-submit pages if the participant drops out or does not
complete the page in time. This should enable the gameplay to proceed
even if there is attrition.

oTree programming For Django Devs
=================================

Intro to oTree for Django developers
------------------------------------

Differences between oTree and Django
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Models
^^^^^^

-  Field labels should go in the template formfield, rather than the
   model field's ``verbose_name``.
-  ``null=True`` and ``default=None`` are not necessary in your model
   field declarations; in oTree fields are null by default.
-  On ``CharField``\ s, ``max_length`` is not required.

Upgrading/reinstalling oTree
============================

There are several alternatives for upgrading or reinstalling oTree.

(TODO: when to use which)

From-scratch reinstallation
---------------------------

-  On Windows: Browse to \`\ ``%APPDATA%`` and delete the folder
   ``otree-launcher``
-  On Mac/Linux: Delete the folder ``~/.otree-launcher``
-  Re-download the launcher according to the instructions on
   http://www.otree.org/download/

In-place upgrade
----------------

Start the launcher and click the "terminal" button to get your console.
Then type:

::

    git pull https://github.com/oTree-org/oTree.git master
    pip install -r requirements_base.txt
    python otree resetdb

Note: you may get merge conflicts if you have modified many files.

Upgrade oTree core libraries (minimal option)
---------------------------------------------

Start the launcher and click the "terminal" button to get your console.
Then type:

Modify ``otree-core`` version number in ``requirements_base.txt`` (the
latest version is
`here <https://github.com/oTree-org/oTree/blob/master/requirements_base.txt>`__),
then run:

::

    pip install -r requirements_base.txt

.. |image0| image:: http://i.imgur.com/Sz34h7d.png
.. |image1| image:: http://i.imgur.com/BtG8ZHX.png
