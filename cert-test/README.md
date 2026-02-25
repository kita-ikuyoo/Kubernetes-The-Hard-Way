# Check for CN and SAN

## Purpose
When connecting to a server with https, if CN and SAN in the corresponding certificate dont include domain or IP used in the command, then the connection will fail. E.g. CN and SAN only contains kube-proxy, but we access the port using 127.0.0.1.

## What to do 
### Bad Case
First generate bad.key and bad.crt.
```
openssl req -x509 -noenc -days 365 -newkey rsa:2048   -keyout bad.key -out bad.crt   -subj "/CN=kube-proxy"
```
Then check the information in bad.crt. You can see that CN=kube-proxy and there is no SAN.
```
openssl x509 -in bad.crt -text -out -bad.text
cat bad.text
```
After that, open a server using bad.key and bad.crt and access it using curl.
```
openssl s_server -key bad.key -cert bad.crt -accept 8443 -www &
curl --cacert bad.crt https://127.0.0.1:8443
```
It will throw the error like `curl: (60) SSL: certificate subject name 'kube-proxy' does not match target host name '127.0.0.1'` which means that it fails to detect `127.0.0.1` from SAN and CN.

### Good Case
First stop the existing server.
```
kill %1
```
Then generate good.key and good.crt.
```
openssl req -x509 -noenc -days 365 -newkey rsa:2048 -keyout good.key -out good.crt -subj "/CN=kube-proxy" -addext "subjectAltName = IP:127.0.0.1,DNS:kube-proxy"
```
Check what is written in good.crt. You will see something like `X509v3 Subject Alternative Name: IP Address:127.0.0.1, DNS:kube-proxy`
```
openssl x509 -in good.crt -text -out good.text
cat good.text
```
Then, open a server using good.key and good.crt and access it using curl.
```
openssl s_server -key good.key -cert good.crt -accept 8443 -www &
curl --cacert good.crt https://127.0.0.1:8443 -o curlgood.text
```
# Summary
SAN and CN are used to match the IP or domain name in curl command.