:warning: This version requires crowdsec version >= 1.5 :warning:

## Auditd collection

A collection for auditd:
 - auditd log parser
 - post exploitation behavior detection

### Backdoor initial execution detection

 - detect a process calling `wget`/`curl` and executing the download script/binary very quickly after
 - detect invocation of obfuscated payloads (ie. `base64 decode | interpreter`)
 - detect invocation of scripts/binaries from hidden directories

 - detect backdoors trying to "kill competitors" :
   - repeated/fast invokation of `rm` 
   - repeated/fast invokation of `kill` / `pkill`

## Acquisition template

Example acquisition for this collection :

```yaml
filenames:
  - /var/log/audit/*.log
labels:
  type: auditd
```

## Warning

The scenario's effectiveness relies on your `auditd` configuration logging EXECVE :

```bash
auditctl -a exit,always -F arch=b64 -S execve
auditctl -a exit,always -F arch=b32 -S execve
```

As an example, you canrestrict it to a specific uid (ie `-F euid=33`) if you want to monitor only your web server.

As an example, see [Florian Roth's example auditd config](https://github.com/Neo23x0/auditd/blob/master/audit.rules).

## Notification template

To have nice-looking notifications on auditd alerts, you can use the following format template:

```yaml
format: |
  {{ range . -}}
  {{$alert := . }}
  *{{$alert.Scenario}}*
  {{ range .Events }}
  `{{.GetMeta "exe"}}` invoked by parent process `{{.GetMeta "parent_progname"}}` (uid={{.GetMeta "uid"}})
  {{ end -}}
  {{- end }}
```
