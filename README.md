# kanidm-k8s

This repo is a work in progress getting kanidm working in k8s.

Read [kanidm docs/install the server](https://github.com/kanidm/kanidm/blob/master/kanidm_book/src/installing_the_server.md), build from there.

## secrets

no certs, secrets in git, format here, fill in the blanks:

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: {{ .Release.Name }}-certs
type: Opaque
data:
 ca.pem:
 cert.pem:
 key.pem:
```
