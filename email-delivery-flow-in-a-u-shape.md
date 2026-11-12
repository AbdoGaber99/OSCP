# Email delivery flow in a U shape

Email path from Ahmed down the left side, across the bottom, and up the right side to Mohamed's MUA.

{% stepper %}
{% step %}
### Ahmed

**ahmed@company.com**

**MUA**\
Mail User Agent
{% endstep %}

{% step %}
### MSA

**Submission Agent**

SMTP 587, AUTH + TLS
{% endstep %}

{% step %}
### Outgoing MTA

**Mail Transfer Agent**
{% endstep %}

{% step %}
### DNS MX lookup

**gmail.com**
{% endstep %}

{% step %}
### Receiving MTA

**Gmail, SMTP port 25**

**Authentication**\
SPF / DKIM / DMARC
{% endstep %}

{% step %}
### MDA

**Mail Delivery Agent**
{% endstep %}

{% step %}
### Mailbox

**mohamed@gmail.com**
{% endstep %}

{% step %}
### MUA

**Gmail web / mobile**
{% endstep %}
{% endstepper %}
