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

Watch those trailing spaces! Hit a snag with the DNS name and weird errors!

Once I had that filled out, I tested connection, synchronized, and then headed over to the Groups configuration panel. There, I selected all the available groups in this server(mathematicians, italians, chemists, etc) and linked them.
Once linked, I configured each one, turning on "@ mentioning" and adding each of my available channels to the group.

You can see my successful @ mention in the Town Squre channel here:

https://postimg.cc/Pv8C0z7C

#

    Create a channel and send a message to it using an incoming webhook. You can use a 
    service such as GitHub to send the webhook or manually send it using curl.

In the Mattermost console, I created a new channel ("webhook-channel" ). Then, in the system settings, I created a new incoming webhook link, assisgned specifically to this channel. To test, I used the following command:

_"curl -i -X POST -H 'Content-Type: application/json' -d '{"text": "Hello! Please give me this job. :D 🎉"}' http://18.116.62.173:8065/hooks/c3ktf7m7cidemjojnpox5tgfyy"_

You can see the result here:

https://postimg.cc/VJY1nk4p

:)

#

    In this task, we are simulating a component of a common pattern encountered by Support Engineers 
    during performance-related incidents. Using whatever means available, find the underlying SQL query that 
    produces results for the “Total Sessions” panel under “System Console > Site Statistics”

So, I dug into this a few ways. One, obviously not SQL, but I looked at the Mattermost Enterprise public Github, dug around into the code for the status dashboard, and found this function call to get the status:

_getStatValue(stats[StatTypes.TOTAL_SESSIONS])_

Next, I thought about it from a system level. The Mattermost service is sending a SQL query to the DB to get returned data (manipulated or not), for these various stats. I took down the containers and edited the Dockerfile (included in assets) to turn on Postgres query logging:

    command: ["postgres", "-c", "log_statement=all"]

Then, I deployed the containers again and started watching the logs for the postgres container after a refresh of the dashboard: 

    sudo docker logs --since 30s 19cd4115a80c

Finally, I saw all the queries. Nearest I can tell, they're executed in left to right order, but it's hard to tell the EXACT status query. I found one, but I'm pretty sure it's for active users. But it's close by. I've included the log file I generated as well. It's somewhere in here:

        024-09-28 03:33:56.637 UTC [36] DETAIL:  parameters: $1 = '0'
        2024-09-28 03:33:56.638 UTC [39] LOG:  execute <unnamed>: SELECT COUNT(*) FROM Status AS s LEFT JOIN Bots ON s.UserId = Bots.UserId LEFT JOIN Users ON s.UserId = Users.Id WHERE LastActivityAt > $1             AND Bots.UserId IS NULL AND Users.DeleteAt = 0
        2024-09-28 03:33:56.638 UTC [39] DETAIL:  parameters: $1 = '1727408036637'
        2024-09-28 03:33:56.653 UTC [39] LOG:  statement: SELECT COUNT(*) FROM Users AS u LEFT JOIN Bots ON u.Id = Bots.UserId WHERE u.DeleteAt = 0 AND Bots.UserId IS NULL
        2024-09-28 03:33:56.653 UTC [36] LOG:  statement: SELECT COUNT(*) FROM Users AS u LEFT JOIN Bots ON u.Id = Bots.UserId WHERE u.DeleteAt = 0 AND Bots.UserId IS NULL
        2024-09-28 03:33:56.654 UTC [34] LOG:  statement: SELECT COUNT(*) FROM Users AS u LEFT JOIN Bots ON u.Id = Bots.UserId WHERE u.DeleteAt = 0 AND Bots.UserId IS NULL
        2024-09-28 03:33:56.655 UTC [39] LOG:  execute <unnamed>: SELECT
                                        TO_CHAR(DATE(TO_TIMESTAMP(Posts.CreateAt / 1000)), 'YYYY-MM-DD') AS Name, Count(Posts.Id) AS Value
                                FROM Posts WHERE Posts.CreateAt <= $1
                                            AND Posts.CreateAt >= $2
                                GROUP BY DATE(TO_TIMESTAMP(Posts.CreateAt / 1000))
                                ORDER BY Name DESC
                                LIMIT 30
        2024-09-28 03:33:56.655 UTC [39] DETAIL:  parameters: $1 = '1727481599999', $2 = '1724716800000'
        2024-09-28 03:33:56.659 UTC [43] LOG:  statement: SELECT COUNT(*) FROM Users AS u LEFT JOIN Bots ON u.Id = Bots.UserId WHERE u.DeleteAt = 0 AND Bots.UserId IS NULL
        2024-09-28 03:33:56.661 UTC [34] LOG:  execute <unnamed>: SELECT
                                        TO_CHAR(DATE(TO_TIMESTAMP(Posts.CreateAt / 1000)), 'YYYY-MM-DD') AS Name, COUNT(DISTINCT Posts.UserId) AS Value
                                FROM Posts WHERE Posts.CreateAt >= $1 AND Posts.CreateAt <= $2
                                GROUP BY DATE(TO_TIMESTAMP(Posts.CreateAt / 1000))
                                ORDER BY Name DESC
                                LIMIT 30
        2024-09-28 03:33:56.661 UTC [34] DETAIL:  parameters: $1 = '1724716800000', $2 = '1727481599999'
        2024-09-28 03:33:56.662 UTC [43] LOG:  execute <unnamed>: SELECT COUNT(*) AS Value FROM Posts p WHERE p.Hashtags <> $1
        2024-09-28 03:33:56.662 UTC [43] DETAIL:  parameters: $1 = ''
        2024-09-28 03:33:56.662 UTC [43] LOG:  execute <unnamed>: SELECT COUNT(*) FROM Commands WHERE DeleteAt = $1

I'm certain if I had more familiary to grep exactly, I'd be able to pinpoint it. :)
