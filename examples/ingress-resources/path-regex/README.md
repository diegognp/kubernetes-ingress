# Support for path regular expressions

NGINX and NGINX Plus support regular expression modifiers for [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)
 directive.

The NGINX Ingress Controller provides the following annotations for configuring regular expression support:

- Optional: ```nginx.org/path-regex: "case_sensitive"``` - specifies a preceding regex modifier to be case sensitive (`~*`).
- Optional: ```nginx.org/path-regex: "case_insensitive"``` - specifies a preceding regex modifier to be case sensitive (`~`).
- Optional: ```nginx.org/path-regex: "exact"``` - specifies exact match preceding modifier (`=`).

[NGINX documentation](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/#nginx-location-priority) provides
additional information about how NGINX and NGINX Plus resolve location priority.
Read [it](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/#nginx-location-priority) before using
 the ``path-regex`` annotation.

Nginx uses a specific syntax to decide which location block to use to handle a request.
Location blocks live within server blocks (or other location blocks) and are used to decide how to process
 the request URI, for example:

```bash
location optional_modifier location_match {
  ...
}
```

The ``location_match`` defines what NGINX checks the request URI against. The existence or nonexistence of the modifier
 in the example affects the way that the Nginx attempts to match the location block.
 The modifiers you can apply using the ``path-regex`` annotation will cause the associated location block
  to be interpreted as follows:

- **no modifier** : No modifiers (no annotation applied) - the location is interpreted as a prefix match.
This means that the location given will be matched against the beginning of the request URI to determine a match

- **~** : Tilde modifier (annotation value ``case_sensitive``) - the location is interpreted as a case-sensitive
regular expression match

- **~***: Tilde and asterisk modifier (annotation value ``case_insensitive``) - the location is interpreted
as a case-insensitive regular expression match

- **=** : Equal sign modifier (annotation value ``exact``) - the location is considered a match if the request
 URI exactly matches the location provided.

## Example 1: Case Sensitive RegEx

In the following example you enable path regex annotation ``nginx.org/path-regex`` and set its value to `case_sensitive`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    nginx.org/path-regex: "case_sensitive"
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea/[A-Z0-9]
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee/[A-Z0-9]
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

Corresponding NGINX config file snippet:

```bash
...

  location ~ "^/tea/[A-Z0-9]" {

    set $service "tea-svc";
    status_zone "tea-svc";

...

  location ~ "^/coffee/[A-Z0-9]" {

    set $service "coffee-svc";
    status_zone "coffee-svc";

...
```

## Example 2: Case Insensitive RegEx

In the following example you enable path regex annotation ``nginx.org/path-regex`` and set its value to `case_insensitive`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    nginx.org/path-regex: "case_insensitive"
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea/[A-Z0-9]
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee/[A-Z0-9]
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

Corresponding NGINX config file snippet:

```bash
...

  location ~* "^/tea/[A-Z0-9]" {

    set $service "tea-svc";
    status_zone "tea-svc";

...

  location ~* "^/coffee/[A-Z0-9]" {

    set $service "coffee-svc";
    status_zone "coffee-svc";

...
```

## Example 3: Exact RegEx

In the following example you enable path regex annotation ``nginx.org/path-regex`` and set its value to `exact` match.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    nginx.org/path-regex: "exact"
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea/
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee/
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

Corresponding NGINX config file snippet:

```bash
...

  location = "/tea" {

    set $service "tea-svc";
    status_zone "tea-svc";

...

  location = "/coffee" {

    set $service "coffee-svc";
    status_zone "coffee-svc";
...
```
