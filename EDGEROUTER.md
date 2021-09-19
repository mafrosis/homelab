Edgerouter
==========

Personal setup plus a couple of tricks.


Global
----------

Disable the Ubiquiti call home:
```
set system analytics-handler send-analytics-report false
set system crash-handler send-crash-report false
```

Set the global domain:
```
set system domain-name eggs
```

Set the router timezone:
```
set system time-zone Australia/Melbourne
```

Disable Ubiquiti UNMS (UISP) management service:
```
set service unms disable
```
