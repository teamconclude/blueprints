# Conclude Blueprints

Blueprints are essential to <a href="https://concludeapp.com"> Conclude</a>.
They let teams implement workflows and processes directly in Slack.

Here are some examples workflows that Conclude supports out of the box:
- Responding to helpdesk inquires
- Managing IT incidents
- Processing recruitment candidates

A blueprint specifies the information content in a certain workflow.
The goal of this document is to teach you how to create your own custom blueprints.

Blueprints are expressed in JSON. You don't need to be a hard core developer to make
your own blueprints, as long as you start out with some existing blueprints and try
and err in your own Slack channels before rolling out new workflows to the entire
enterprise.

## Key Concepts

- A **blueprint** contains meta information for a workflow of a certain kind.
- An **activity** is an instance of a blueprint, in the same way that a cookie
is an instance of a cookie cutter. Or creating a helpdesk ticket (= activity) from
the helpdesk blueprint.
- An **activity channel** is a dedicated, ephemeral Slack channel created by Conclude when
an activity is created. The purpose of the channel is to deal with an an activity
(for example, a helpdesk ticket). The channel is archived when the activity is
concluded (closed).
- A **parent channel** is the Slack channel that an activity would come from, also called
a **target channel** in the context of installing blueprints.

Here's how this works in practice:
- A company has installed the *incident* blueprint in *#ict-incidents*.
- An employee or contractor reports an incident with Conclude using `/c incident`.
- Conclude will create an incident *activity* in the #_ict-incidents-103 channel.
- \#ict-incidents is the parent channel and #_ict-incidents-103 is the activity channel.

## The Basic Blueprint

The *basic* (default) blueprint in Conclude consists of three attributes:
- A mandatory *title* that contains the subject of a basic activity.
- An optional *body* field that contains a detailed text.
- An optional *conclusion* field with the conclusion text.

````json
{
  "name": "basic",
  "type": "basic",
  "description": "The default blueprint",
  "attributes": [
    {
      "name": "title",
      "type": "string",
      "label": "Subject",
      "placeholder": "Enter subject here",
      "mandatory": true
    },
    {
      "name": "body",
      "type": "text",
      "label": "Details",
      "placeholder": "Enter detailed description here"
    },
    {
      "name": "conclusion",
      "type": "text",
      "label": "Conclusion",
      "placeholder": "Enter conclusion here"
    }
  ]
}    
````

## Blueprint Name, Type and Scope

A blueprint has a *name* and a *type*. The name can contain the same characters
as a Slack channel; lower case letters, numbers, periods, and underscores. The type
must be one of the following:
- **basic**: The basic blueprint.
- **standard**: A standard blueprint.
- **custom**: A custom blueprint with semi-global scope.
- **global**: A custom blueprint with global scope.

The main difference between the types is scope; i.e. who can use the blueprint, and
how is it invoked.

To use a *basic* or *standard* blueprint, the user must enter the channel where the
blueprint is installed and type `/c new` to create an activity based on the blueprint.
The blueprint is *invoked locally* from inside the channel, and the user will of course
need to be a member of the channel to use the blueprint.

Blueprints of *custom* and *global* type can be invoked with the `/c <blueprint>`
command from anywhere in Slack. The user does not need to enter into the channel
where the blueprint was installed (for example in `#ict-helpdesk`) but can run the
`/c helpdesk` from anywhere.

### Custom Blueprints = Restricted Scope

A *custom* blueprint is limited to users who are members of the channel where the
blueprint was installed, while *global* has no such limitations and can even be used
by multi-channel guests, such as external contractors.

You would normally use *custom* blueprints to implement workflows for functional or
geographical units of the organization, and *global* blueprints for company wide use.

### Global Blueprints: Company Wide Scope

Global blueprints are useful, because they let anyone in the workspace use a blueprint
to file an incident report, submit a helpdesk ticket etc., however, they are not
permitted to see the "inside" of the incident or helpdesk activity unless you give
them an explicit invitation.

You can think of global blueprints as a way to build a wall of separation between the
people on the outside who file a report, create a ticket etc. and the internal team on
the inside. 

In order to make global blueprints work, you will need to specify who is on the
inside team, and who will be notified about the new activity.

This is exactly what the `/c blueprint install` command does.

## Default Membership and Notifications

A blueprint can specify the default owner and members (the internal team), and how to notify
about the creation and conclusion of activities. The default behavior (when using the basic
blueprint) is:
- The user who created the activity with `/c new` becomes the owner.
- No members are invited by default.
- No notifications are sent out, except that the owner sees a new activity channel.

It is then up to the owner to invite members to the activity by inviting them through Slack.  The
`/c invite` command is a convenient way to invite several users or user groups to an activity.

When someone installs a blueprint with the `/c blueprint install` command, Conclude will
use these default settings (except when setting the basic blueprint):
- The type is set to *global* so the blueprint becomes available to anyone in the team.
- The person who installed the blueprint becomes the default owner of all activities created with this blueprint.
- The channel members of the blueprint's install channel will automatically be invited to all activities. 
- Notifications will be posted in the install channel when an activity is created and concluded. 

You may modify these settings using `/c blueprint edit` to update the JSON format directly, or use
Conclude commands to change these settings.  You might see something like this:
````json
{
  "name": "incident",
  "type": "global",
  "owner": "@patricia:U12Q150H2",
  "members": "#ict-incidents:CKRJ2EF59",
  "notify": "#ict-incidents:CKRJ2EF59",
````

The format is @slackuser:USER_ID or @slackusergroup:USERGROUP_ID, or #channel:CHANNEL_ID.
Conclude only uses the ID after the colon. The user- og channel name before the colon is just
for readability purposes.

We recommend that you never edit these settings directly in the JSON file but instead use
the following commands to set them:
- `/c blueprint set owner @patricia`
- `/c blueprint set members #ict-incidents` 
- `/c blueprint set notify #incidents-log @ict-team` 

Conclude will look up the actual Slack IDs and insert them into the JSON.

The **owner** must be a Slack team member that has permissions to create a new Slack channel.
This user will become the owner of all activities creates with the blueprint. Conclude will
keep track of the user who initiated the activity and display this information at the top of
the activity. As noted above, if no owner has been set, the user who creates the activity
will become the initial owner (and initiator).

**TIP:** If you want the activity to be unassigned to begin with, create a Slack user called
unassigned and make this the default owner.

You may change the owner later by using the `/c owner` command, or directly from the
settings menu in the Conclude Inbox.

The **members** setting can include any combination of users, user groups and channels.
Setting a channel as a member means inviting the current channel members to the activity.
Think of the channel as a Slack user group, i.e. a list of people to be invited.

You may invite more members later, by using the `/c invite` command, or through Slack.

The **notify** setting takes the same parameters as the members setting. It tells Conclude
whom to notify when an activity has been created, concluded or re-opened.

**TIP**: Some teams prefer to send all notifications to a place: `/c blueprint set notify #conclude-logs`.   

The only way to change the notification setting is to change it in the blueprint.

Conclude can also show or clear settings:
- `/c blueprint show members`
- `/c blueprint clear notify`

Don't remember the blueprint command later? No problem. Just type `/c help blueprint`.
to

## Attributes

A blueprint can have up to 10 attributes, which is the maximum number of dialog elements
supported by Slack.

### Core Properties

An attribute has a mandatory *name*, *label* and *type*, and several other properties
described later:
- **name**: The attribute internal name.
- **type**: The attribute type, explained below.
- **label**: A visible label displayed before the attribute.

The attribute types can be one of the following:
- **string**: A single line text up to 150 characters.
- **text**: A multi-line text area up to 3k characters.
- **select**: A select dropdown with up to 100 choices.
- **user**: A special select dropdown to select a Slack user.
- **link**: A special single line text to store a URL.

### Predefined Attributes

There are several predefined attributes. They may be customized with a new label, but
their type must remain the same:
- **title** (*string*): The main subject headline identifies the activity and is always required.
- **body** (*text*): A general purpose text area (up to 3k characters).
- **conclusion** (*text*): The conclusion should contain a closing summary of the activity.
- **deadline** (*text*): Activity deadline (implicit attribute)
- **severity** (*select*): Activity severity (implicit attribute).

Implicit attributes are not part of the blueprint even if they are not in the attribute list.
They are also normally not displayed in the input dialog where a user creates a new activity.

### More Attribute Properties

In addition to the obligatory attributes *name*, *title* and *label* described above,
there are a few other properties:

- **mandatory** (boolean): Setting *mandatory* to *true* (not "true") means that the
attribute should always have a non-empty value. This is the case for the *title* attribute.
- **default_value**: The default value of an attribute.
- **placeholder**: A text displayed inside the input field before the user starts typing.
- **hint**: A contextual hint about the format etc. of the attribute.
- **visible**: Controls where the attribute is displayed (default = "").

The **visible** setting can be:
- empty: This is the default setting. The attribute will be shown in all dialogs.
- "create": The attribute will only be shown in the create dialog.
- "edit": The attribute will only be shown in the conclude dialog.
- "conclude": The attribute will only be shown in the conclude dialog.
- "create,edit": Shown in the create and edit dialogs but not in the conclude dialog.

Here's how to add an attribute that's visible internally but not to the person who
creates the activity:
````json
{
  "name": "notes",
  "type": "string",
  "label": "Internal notes",
  "placeholder": "Internal investigation notes",
  "visible": "edit,conclude"
} 
````

## Further Blueprint Customization

A blueprint offers a number of ways to customize the visual presentation and
user experience of a workflow.

The two *disable_...* settings take boolean values and should be set to 
true or false (not "true" or "false"):
- **disable_deadline**: Can be *true* or *false* (default). If *disable_deadline* is
set Conclude will hide the deadline settings from the user interface.
- **disable_severity**: Can be *true* or *false* (default). If *disable_severity* is
set Conclude will hide the severity settings from the user interface.

You can change the dialog caption (title) with these settings:
- **create_caption**: Sets the caption of the dialog where the user creates a new
activity. The default caption is "New Activity", or "New <blueprint label>"
if the blueprint has a label setting.
- **edit_caption**: Sets the caption of the dialog where the team edits an
existing activity. The default is "Edit Activity" or "Edit <blueprint label>".
- **conclude_caption**: Sets the caption of the dialog where the activity is
concluded. The default is "Conclude Activity" or "Conclude <blueprint label>".

````json
  "create_caption": "Report New Incident"
````

These settings modify the text of the OK button in the dialogs just described:
- **create_button**: Sets the text of the OK button in the create dialog (default: "Submit").
- **edit_button**: Sets the text of the OK button in the edit dialog (default: "Save").
- **conclude_button**:  Sets the text of the OK button in the conclude dialog (default: "Done").

````json
  "conclude_button": "Close Incident"
````

You may also customize the notification that is sent to the blueprint members:

- **create_intro**: The default text ":wave: New activity" or ":wave: New \<label\>"
if the blueprint has a label setting.
- **conclude_intro**: The default text is ":white_check_mark: Concluded activity"
or ":white_check_mark: Concluded \<label\>".
