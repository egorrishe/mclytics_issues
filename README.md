# Quick Start
1. login http://admin.vangoghexhibit.ca/
2. **Email Template**:
    1. build email template via BeeFree (or etc.)
    2. open [Email Templates](http://admin.vangoghexhibit.ca/email-tpl)
    3. [Create new one](http://admin.vangoghexhibit.ca/email-tpl/create)
    4. in field `body` insert HTML of email. 
    5. Fill other fields, if necessary.
    6. Enable `Is Default` checkbox - this template will be used for all [Ticket Events](http://admin.vangoghexhibit.ca/ticket-event)
    7. Save. And you will see preview of email.
3. **Settings**
    * **NOTE:** now cron is disabled. To enable it do next:
        1. open [Settings](http://admin.vangoghexhibit.ca/settings)
        2. check all, config as you need.
        3. enable setting [email_cron_enable](http://admin.vangoghexhibit.ca/settings/update?id=4) 
4. That's all  

**Additionally**:
* To each [Ticket Event](http://admin.vangoghexhibit.ca/ticket-event) you can assign custom [Email Template](http://admin.vangoghexhibit.ca/email-tpl):
    1. Open [Ticket Events](http://admin.vangoghexhibit.ca/ticket-event)
    2. use filter to find exact Event
    3. click update button (pencil). (e.g. http://admin.vangoghexhibit.ca/ticket-event/update?id=114)
    4. Choose Email Template, and Save.
* you can click each item in top Menu bar - and investigate all features. I guess they all are clear, without instructions. If any questions - ask me.

# Report

## Webhook receiver:
* I don't know - can event's date be changed? Just in case it can - each time Script will check "is date changed?" and if YES - update it
* script stores Customer's First Name, Last Name. 
    * I don't know will you use it, or not - I store it just in case...
    * If later Customer will change some of these fields, script will detect it and update this info.
        * Additionally Universe provide other Customer's info (e.g. address, phone number). If you need it - later we can save it too. 
        * You can see whole webhook data -  http://admin.vangoghexhibit.ca/webhook-log/view?id=2072.
        
## Cron Jobs
* cron is running once per 5 minutes.
    * You can change frequency in cPanel: https://prnt.sc/v7vojk -> https://prnt.sc/v7vp6r
* Each cron job logs in DB - you can see it here [Cron Logs](http://admin.vangoghexhibit.ca/cron-log)
        
## Logs:
* If any **error** happen:
    * email will be sent:
        * to - greg@starvoxent.com
        * from - logger@admin.vangoghexhibit.ca
        * subject - `Error log message. No reply.`
            * so you can create rule in your mailbox to organize such mails, and to avoid them from spam filter
    * additional email will be sent to me
    * also all errors related to webhook, will be duplicated into file `/home/coreyr13/public_html/admin/backend/runtime/logs/universe_webhook_error.log`
    * also all errors related to cron, will be duplicated into file `/home/coreyr13/public_html/admin/backend/runtime/logs/cron/TicketEventMailer.log`
    * all other errors will be duplicated into file `/home/coreyr13/public_html/admin/backend/runtime/logs/app.log`
* Any `Webhook` request logs into:
    * file `/home/coreyr13/public_html/admin/backend/runtime/logs/universe_webhook_info.log` (it's for debugging)
    * DB `coreyr13_db_admin_ticket.webhook_log`
    
## Notes
1. Imagine example:
    * Customer buy 1 ticket.
    * Our website save him to DB.
    * Later, he decided to buy 2nd ticket to the same event (e.g. for his friend).
    * Our website will skip him - because he already linked to this event.
    * Before 24hrs user will receive email with suggestion to buy 1 upgrade.
    * But really he can buy 2 upgrades (because he has 2 tickets).
        * So you should provide him such possibility in your TitanForm email.
        * If you want to improve script to operate with such cases somehow - let me know. 
            * Webhook Logs: [#9](http://admin.vangoghexhibit.ca/webhook-log/view?id=9) & [#10](http://admin.vangoghexhibit.ca/webhook-log/view?id=10) are the real example of this case.

# Development Installation
* [Yii2](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide/start-installation.md)
* DB: `CREATE DATABASE admin_vangoghexhibit DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;`


# DB statistics:
1. In CPanel find and click button `phpMyAdmin`.
2. [Choose DB and open SQL tab](https://prnt.sc/vcezh3)
3. Examples of statistics:
    * *Average execution time of cron jobs*:
        * ```mysql
            SELECT ROUND(MIN(cj.last_execution_time), 2) `MIN(sec)`, 
                   ROUND(MAX(cj.last_execution_time), 2) `MAX(sec)`, 
                   ROUND(AVG(cj.last_execution_time), 2) `AVG(sec)`
            FROM cron_job cj
                JOIN ticket_event_to_cron_job te2cj ON cj.id_cron_job = te2cj.cron_job_id;
          ```
    * *Average count of emails **per HOUR***:
        * ```mysql
            SELECT MIN(n), MAX(n), ROUND(AVG(n)) as `AVG(n) per HOUR`
            FROM
            (
                SELECT COUNT(*) as n
                FROM cron_job cj
                    JOIN ticket_event_to_cron_job te2cj ON cj.id_cron_job = te2cj.cron_job_id
                WHERE cj.id_cron_job <> 1 /*exclude 1st cron. We should analyze only regular cron jobs*/
                GROUP BY ROUND(cj.started_at / 3600)
            ) as per_hour;
          ```
    * *Average count of emails **per DAY***:
        * ```mysql
            SELECT MIN(n), MAX(n), ROUND(AVG(n)) as `AVG(n) per DAY`
            FROM
            (
                SELECT COUNT(*) as n
                FROM cron_job cj
                    JOIN ticket_event_to_cron_job te2cj ON cj.id_cron_job = te2cj.cron_job_id
                WHERE cj.id_cron_job <> 1 /*exclude 1st cron. We should analyze only regular cron jobs*/
                GROUP BY ROUND(cj.started_at / 86400)
            ) as per_day;
          ```
          
