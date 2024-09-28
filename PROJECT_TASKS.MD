Deployment (See deployment guide)


For the sake of ease, I used the preview docker. For LDAP, I tried using several LDAP installations and kept running into weird connection errors (probably something simple) I couldn't quite figure out (manual connection always worked but Mattermost wouldn't). For ease, I used an online test LDAP server with users and groups with Mattermost (https://www.forumsys.com/2022/05/10/online-ldap-test-server/).

ldap.forumsys.com
389
password
base dn: dc=example,dc=com
uid
uid
mail

Synchronized groups in ldap server - mathematicians, italians, chemists, etc. @ mentioned them in the town square channel

Incoming Webhook

Created channel "webhook-channel" in my local isntallation
Created incoming webhook in settings, configured for that channel

curl -i -X POST -H 'Content-Type: application/json' -d '{"text": "Hello! Please give me this job. :D 🎉"}' http://localhost:8065/hooks/joze5qowxt8g7pscezzzfef3he

Incident Simulation Task

Still working with the postgres container to get it logging all queries (turning logging_statement to 'all' but not getting anything?), but I dug into the source code and found the line that triggers the query:

getStatValue(stats[StatTypes.TOTAL_SESSIONS]
