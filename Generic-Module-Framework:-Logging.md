
The generic module framework supports simple logging to show how patients progress through modules.  When enabled, a table will be printed to standard out when each patient reaches a `Terminal` state in the workflow.  _(Note that if a patient never reaches a `Terminal` state, then that patients table will not be generated)._

Logging is disabled by default.  To enable logging, turn it on in the _config/synthea.yml_ file:
```yaml
  generic:
    log: true
```

**Examples From the Examplitis Module**

The following example shows the log of a patient who was female, so she went from "Initial" directly to "Terminal".

```
/===============================================================================
| Examplitis Log
|===============================================================================
| Entered                   | Exited                    | State
|---------------------------|---------------------------|-----------------------
| 1980-02-02T19:59:42-05:00 | 1980-02-02T19:59:42-05:00 | Initial
| 1980-02-02T19:59:42-05:00 |                           | Terminal
\===============================================================================
```

As opposed the the previous example, this example shows the log of a patient who acquired Examplitis and ultimately needed surgery.  Since he went from "Examplotomy" to "Terminal", however, we know he did not _die_ of Examplitis.

```
/===============================================================================
| Examplitis Log
|===============================================================================
| Entered                   | Exited                    | State
|---------------------------|---------------------------|-----------------------
| 1957-06-28T18:44:14-04:00 | 1957-06-28T18:44:14-04:00 | Initial
| 1957-06-28T18:44:14-04:00 | 1998-06-26T18:44:14-04:00 | Age_Guard
| 1998-06-26T18:44:14-04:00 | 2005-06-26T18:44:14-04:00 | Pre_Examplitis
| 2005-06-26T18:44:14-04:00 | 2005-06-26T18:44:14-04:00 | Examplitis
| 2005-06-26T18:44:14-04:00 | 2005-08-30T00:54:57-04:00 | Wellness_Encounter
| 2005-08-30T00:54:57-04:00 | 2005-08-30T00:54:57-04:00 | Examplitol
| 2005-08-30T00:54:57-04:00 | 2008-07-30T00:54:57-04:00 | Pre_Examplotomy
| 2008-07-30T00:54:57-04:00 | 2008-07-30T00:54:57-04:00 | Examplotomy_Encounter
| 2008-07-30T00:54:57-04:00 | 2008-07-30T00:54:57-04:00 | Examplotomy
| 2008-07-30T00:54:57-04:00 |                           | Terminal
\===============================================================================
```