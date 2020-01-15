# k8s

If everyone else jumped off a cliff, would you?

Apparently, yes.



```
root@mac:~/k8s# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-rmm8j
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 966b8051-477f-415a-b206-6f92220bfc5a

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkpLTm9jSWtRV25wZDJWSEFzNEhuRk9fZE93TU5HYnBKTC05VVM2MEszWDgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXJtbThqIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI5NjZiODA1MS00NzdmLTQxNWEtYjIwNi02ZjkyMjIwYmZjNWEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.qhC-l0fQBCJG8j-VjnivRnyIAJAsq8orI9ZPWPEtlCXbaX3A9J9W71XLM6il0NUdWV8rtJGVlK2gzbNDInQKxFMLM5obNDv0A1P4lByuIQKqHyqh1KgyYJHbvod9PY1Vv-YHJ40-w7oSM1HHhlBrmp-MBECSNfTDJ5BCmXE0zYbXxwOtKH7K-yuicNhvFnyl_est6SMxde-EhsolFDsGBj15v9fw7hk8NU478MVa_TxRmb2ubVCTPRYswiyYu3QcKAyYZ-87y0Bp2xLitF544xxp_zME_5wI59JW6A9urU53pk-XPtyR9NwICrLYV2BiyaY871LeXJ2nIbQPgBxX3A
root@mac:~/k8s# 
```
