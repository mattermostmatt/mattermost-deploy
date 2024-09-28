# Mattermost Interview Project

Please find below, in a step by step fashion, the answers/results of the tasks assigned.


    ● Setup an Enterprise licensed Mattermost server in any of the following deployment
    models:
          ● Mattermost Operator on your K8s environment of choice
          ● Mattermost Preview Docker
          ● Mattermost Production Docker - This one is a bit more involved than the preview
          image but better reflects a deployment you’d see at a customer
          ● You can request an enterprise trial on our website or through the server’s system
          console; alternatively, we can email you a license.

For details on the various deployments I tried (all of them), please see assets/README.MD

All steps below were accomplished using the Mattermost Production Docker, with a license requested through the console (the one with the Mattermost and Postgres standalone containers).

#
    Integrate an LDAP Server and configure at least one group with LDAP Group Sync that 
    you can @-mention within a channel. You can use this Docker image to quickly get
    OpenLDAP running.


So, I tried using several LDAP installations and kept running into weird connection errors (probably something simple) that I couldn't quite figure out. Troubleshooting wise, I tried connecting to openLDAP manually, but had issues making them talk. That being said, and for ease, I went ahead and used a known good online test LDAP server (that I've used before) with sample users and groups to good success. (https://www.forumsys.com/2022/05/10/online-ldap-test-server/).

In the server settings for AD/LDAP, I filled in the following settings and left the rest blank/default:

    AD/LDAP Server: ldap.forumsys.com 
    AD/LDAP Port: 389 
    Bind Password: password 
    Base DN: dc=example,dc=com 
    ID Attribute: uid
    Login ID Attribute: uid
    Username Attribute: uid
    Email Attribute: mail
    Group Display Name Attribute: cn
    Group ID Attribute: entryUUID
    

Once I had that filled out, I tested connection, synchronized, and then headed over to the Groups configuration panel. There, I selected all the available groups in this server(mathematicians, italians, chemists, etc) and linked them.
Once linked, I configured each one, turning on "@ mentioning" and adding each of my available channels to the group.

You can see my successful @ mention in the Town Squre channel here:

https://postimg.cc/VJY1nk4p

#

    Create a channel and send a message to it using an incoming webhook. You can use a 
    service such as GitHub to send the webhook or manually send it using curl.

In the Mattermost console, I created a new channel ("webhook-channel" ). Then, in the system settings, I created a new incoming webhook link, assisgned specifically to this channel. To test, I used the following command:

_"curl -i -X POST -H 'Content-Type: application/json' -d '{"text": "Hello! Please give me this job. :D 🎉"}' http://localhost:8065/hooks/joze5qowxt8g7pscezzzfef3he"_

You can see the result here:

https://postimg.cc/VJY1nk4p

:)

#

    In this task, we are simulating a component of a common pattern encountered by Support Engineers 
    during performance-related incidents. Using whatever means available, find the underlying SQL query that 
    produces results for the “Total Sessions” panel under “System Console > Site Statistics”

Still working with the postgres container to get it logging all queries (turning logging_statement to 'all' but not getting anything?), but I dug into the source code and found the line that triggers the query:

_getStatValue(stats[StatTypes.TOTAL_SESSIONS])_
