# Deploy
🚧 This guide is currently WIP 🚧

!!Currently, the Keycloak example is the only functional setup!!

## Prerequesites
- Docker capabilities*
- k8s cluster (for example minikube)

## Deploy operator
```helm upgrade --install operator confluentinc/confluent-for-kubernetes --namespace confluent --version 0.1033.3```

```
kubectl create -n confluent secret generic oauth-jass --from-file=oauth.txt=oauth_jass.txt 
kubectl -n confluent create secret generic oauth-jass-oidc --from-file=oidcClientSecret.txt=oauth_jass.txt
```
## Deploy Keycloak (if applicable)

```kubectl apply -f keycloak_deploy.yaml```

## Create Secrets
```
openssl genrsa -out ca-key.pem 2048

openssl req -new -key ca-key.pem -x509 \
  -days 1000 \
  -out ca.pem \
  -subj "/C=US/ST=CA/L=MountainView/O=Confluent/OU=Operator/CN=TestCA"

kubectl create secret tls ca-pair-sslcerts \
  --cert=ca.pem \
  --key=ca-key.pem -n confluent

kubectl create secret generic tls-group1 \
--from-file=fullchain.pem=certs/generated/server.pem \
--from-file=cacerts.pem=certs/generated/ca.pem \
--from-file=privkey.pem=certs/generated/server-key.pem \
-n confluent

kubectl create secret generic credential \
--from-file=plain-users.json=creds-kafka-sasl-users.json \
--from-file=plain.txt=creds-client-kafka-sasl-user.txt \
--from-file=digest.txt=digest.txt \
--from-file=digest-users.json=digest-users.json \
--from-file=oidcClientSecret.txt=oidcClientSecret.txt \
-n confluent

kubectl create secret generic credential \
--from-file=plain-users.json=creds-kafka-sasl-users.json \
--from-file=plain.txt=creds-client-kafka-sasl-user.txt \
--from-file=oidcClientSecret.txt=oidcClientSecret.txt \
-n confluent

kubectl create secret generic mds-token \
--from-file=mdsPublicKey.pem=mds-publickey.txt \
--from-file=mdsTokenKeyPair.pem=mds-tokenkeypair.txt \
-n confluent
```

## Deploy Confluent

```
kubectl apply -f cp_components_entra.yaml
or
kubectl apply -f cp_components_keycloak.yaml
```

## Deploy Rolebindings
```
kubectl apply -f rolebindings.yaml
kubectl apply -f topic-rolebindings.yaml
```

# Control Center Access
To be able to see the cluster in the Control Center, a user or group needs to have at least the "Operator" role. See the Confluent [documentation](https://docs.confluent.io/platform/current/control-center/security/c3-rbac.html#id9:~:text=connections%20are%20proxied.-,Topic%20management%20(cluster%20scope,-)%C2%B6) for the complete set of roles. For example:

```yaml
apiVersion: platform.confluent.io/v1beta1
kind: ConfluentRolebinding
metadata:
  name: g1-operator-kafka
  namespace: confluent
spec:
  principal:
    name: "/g1" # corresponding group from idp against testuser1
    type: group
  role: Operator
```

## Topic Access
With just the Operator role a user can not create or modify any topic they want. To be able to create or access a topic, the user needs to have access to the topic. The rolebinding displayed will give testuser1 the ResourceOwner role on the topic cloud.test.produce. This will allow testuser1 to create, modify, read and write to the given topic.

```yaml
apiVersion: platform.confluent.io/v1beta1
kind: ConfluentRolebinding
metadata:
  name: testuser1.resource-owner
  namespace: confluent
spec:
  principal:
    type: user
    name: 163ee8e8-04f8-4de2-ba22-daf394eea8f1
  role: ResourceOwner
  resourcePatterns:
  - name: cloud.test.produce
    patternType: PREFIXED
    resourceType: Topic
```

## Producer/Consumer access
WIP

