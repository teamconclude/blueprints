# Conclude Blueprints

Blueprints are central in <a href="https://concludeapp.com"> Conclude</a>,
which lets team implement workflows and processes directly in Slack, for example:

- Responding to helpdesk inquires
- Managing IT incidents
- Processing recruitment candidates

A blueprint specifies the information content in a certain workflow.
The goal of this document is to teach you how to create your own custom blueprints.

## Key Concepts in Conclude

- A **blueprint** contains meta information for a workflow of a certain kind.
- An **activity** is an instance of a workflow, for example a helpdesk ticket.
- An **activity channel** is an (ephemeral) Slack channel created by Conclude when an activity
is created, and archived when the activity is concluded (closed).
- A **parent channel** is the Slack channel that an activity would come from, also called
a **target channel** in the context of installing blueprints.

Here's an example:
- Your company has installed the *incident* blueprint in *#ict-incidents*.
- An employee or contractor reports an incident with Conclude using `/c incident`.
- Conclude will create an incident *activity* in the #_ict-incidents-103 channel.
- \#ict-incidents is the parent channel and #_ict-incidents-103 is the activity channel.

## The Basic Blueprint

Blueprints are expressed in JSON. The *basic* (default) blueprint in Conclude
has the name and type "basic", a short description, and contains three fields:
- A mandatory *title* that contains the subject of a basic activity.
- An optional *body* field that contains a detailed text.
- An optional *conclusion* field with the conclusion text.

```
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
```

## Blueprint Name, Type and Scope

A blueprint must have a *name* and a *type*. The name can contain the same characters
as a Slack channel; lower case letters, numbers, periods, and underscores. The type
can be one of the following:
- **basic**: The basic blueprint.
- **standard**: A standard blueprint.
- **custom**: A custom blueprint with semi-global scope.
- **global**: A custom blueprint with global scope.

The main difference between the types is scope; who can use the blueprint, and
how is the blueprint invoked.

To use a *basic* or *standard* blueprint, the user must enter the channel where the
blueprint is installed and type `/c new` to create an activity based on the blueprint.
The blueprint invoked locally from inside the channel, and the user needs to be a
member of the channel where the blueprint is installed.

Blueprints with *custom* and *global* type can be invoked with the `/c <blueprint>`
command directly from anywhere in Slack. The user doesn't need to go to the channel
where the blueprint is installed (for example `#it-helpdesk`) but can run the
`/c helpdesk` from anywhere.

A *custom* blueprint is limited to users who are members of the channel where the
blueprint is installed, while *global* has no such limitations and can even be used
by multi-channel guests, such as external contractors.

The user therefore needs access to the 
globally available team blueprint. This is the type of blueprints that are installed
with the `/c blueprint install command` (except when installing the basic blueprint).

The main advantage with global blueprints is that they can be accessed by anyone on the team,
even if they're not a member of the channel 

## Default Membership and Notifications

A blueprint can specify the default owner and members, and how to notify about the
creation and conclusion of activities. The default behavior (when using the basic
blueprint) is:
- The user who created the activity with `/c new` becomes the owner.
- No members are invited by default.
- No notifications are sent out, except that the owner sees a new activity channel.

It is then up to the owner to invite members to the activity by inviting them through Slack.  The
`/c invite` command is a convenient way to invite several users or user groups to an activity.

When someone installs a blueprint with the `/c blueprint install` command, Conclude will
use these default settings (except when setting the basic blueprint):
- The type is set to *global* so the blueprint becomes available to anyone in the team.
- The default owner is set to the user who installed the blueprint.
- All members of the blueprint's install channel will become members of the activity by default. 
- Notifications will be posted in the install channel when an activity is created and concluded. 

You may modify these settings using `/c blueprint edit` to update the JSON format directly, or use
Conclude commands to change these settings.

This is how it might look in the JSON format:
```
{
  "name": "incident",
  "type": "global",
  ...
  "owner": "@patricia:U12Q150H2",
  "members": "#ict-incidents:CKRJ2EF59",
  "notify": "#ict-incidents:CKRJ2EF59",
  ...
```

The format is @slackuser:USER_ID or @slackusergroup:USERGROUP_ID, or #channel:CHANNEL_ID.
Conclude only uses the ID after the colon. The user- og channel name before the colon is just
for readability purposes.

We recommend that you do not edit these settings in the JSON file but instead use commands to
set them:
- `/c blueprint set owner @patricia`
- `/c blueprint set members #ict-incidents` 
- `/c blueprint set notify #incidents-log @ict-team` 

Conclude will look up the actual Slack IDs and insert them into the JSON.

The **owner** must be a Slack team member that has permissions to create a new Slack channel.
This user will become the owner of all activities creates with the blueprint. Conclude will
keep track of the user who initiated the activity and display it at the top of the activity.
As noted above, if no owner has been set, the user who creates the activity will become the
initial owner (and initiator).

**TIP:** If you want the activity to be unassigned to begin with, create a Slack user called
unassigned and make this the default owner.

You may change the owner later by using the `/c owner` command, or directly from the
settings menu in the Conclude Inbox.

The **members** setting can include any combination of users, user groups and channels.
Setting a channel as a member means inviting the current channel members to the activity.

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
- **label**: A visible label displayed before the attribute.
- **type**: The attribute type; *string*, *text*, *select*, *user* or *link*

### Pre-defined Attributes

There are several pre-defined attributes. They may be customized with a new label, but
their type must remain the same:
- **title** (*string*): The main subject headline identifies the activity and is always required.
- **body** (*text*): A general purpose text area (up to 3k characters).
- **conclusion** (*text*): The conclusion should contain a closing summary of the activity.
- **deadline** (*text*): Activity deadline (implicit attribute)
- **severity** (*select*): Activity severity (implicit attribute).

Implicit attributes are not part of the blueprint even if they are not in the attribute list.
They are also normally not displayed in the input dialog where a user creates a new activity.
