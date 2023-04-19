# Insert 1.000 users (on each running meeting)

```sh
QTT=100 ;sudo -u postgres psql -U postgres -d bigbluebutton <<DOC
          INSERT INTO "user" 
          SELECT substring(md5(random()::text) FROM 1 FOR 14), 
                  "extId", "meetingId", 'User ' || lpad(( (select count(*) from "user" u where u."meetingId" = one_user_of_each_meeting."meetingId")  + i) ::varchar, 5, '0'),
                  avatar, color, emoji, guest, 
                  "guestStatus", mobile, "clientType", "role", 
                  authed, joined, "leftFlag", banned, 
                  "loggedOut", "registeredOn", presenter, pinned, 
                  "locked" 
          FROM (
              select * from public."user" u
              where u."userId" in (select min("userId") FROM public."user" group by "meetingId")
              and exists (select 1 from "meeting" m where m."meetingId" = u."meetingId")
          ) one_user_of_each_meeting, generate_series(1, $QTT) i
DOC
```


# Run it every 1s

```sh
while [ true ] ; do 
  sleep 1; 
  QTT=1000 ; 
  (
        sudo -u postgres psql -U postgres -d bigbluebutton <<DOC
          INSERT INTO "user" 
          SELECT substring(md5(random()::text) FROM 1 FOR 14), 
                  "extId", "meetingId", 'User ' || lpad(( (select count(*) from "user" u where u."meetingId" = one_user_of_each_meeting."meetingId")  + i) ::varchar, 5, '0'),
                  avatar, color, emoji, guest, 
                  "guestStatus", mobile, "clientType", "role", 
                  authed, joined, "leftFlag", banned, 
                  "loggedOut", "registeredOn", presenter, pinned, 
                  "locked" 
          FROM (
              select * from public."user" u
              where u."userId" in (select min("userId") FROM public."user" group by "meetingId")
              and exists (select 1 from "meeting" m where m."meetingId" = u."meetingId")
          ) one_user_of_each_meeting, generate_series(1, $QTT) i
DOC
  ); 
done

```
