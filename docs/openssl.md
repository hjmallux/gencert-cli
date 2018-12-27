## OpenSSL 证书生成命令

```bash
#!/usr/bin/env bash

## ------ library function ------

## ca.pem
__do_openssl_gencert_ca_for_docker() {
	cat >${extfile_dir}/ca.extfile<<-EOF
	[v3_ca]
	keyUsage = critical, digitalSignature, keyEncipherment, keyAgreement, keyCertSign
	basicConstraints = critical, CA:true
	EOF

    openssl req -new \
                -newkey rsa:2048 \
                -days $days \
                -x509 \
                -nodes \
                -subj "/O=root" \
                -keyout ${certs_dir}/ca-key.pem \
                -out ${certs_dir}/ca.pem \
                -config ${conf_dir}/openssl.cnf \
                -extensions v3_docker_ca
    return 0
}

## server.pem
__do_openssl_gencert_server_for_docker() {
	cat >${extfile_dir}/server.extfile<<-EOF
	keyUsage = critical, digitalSignature, keyEncipherment, keyAgreement
	extendedKeyUsage = serverAuth
	basicConstraints = critical, CA:false
	subjectAltName = DNS.1:localhost, IP.1:127.0.0.1, IP.2:${ipaddr}
	EOF

    ## serer-key.pem and server.csr
    openssl req -new \
                -newkey rsa:2048 \
                -days ${days:=7300} \
                -nodes \
                -subj "/O=root.$(hostname)" \
                -keyout ${certs_dir}/server-key.pem \
                -out ${certs_dir}/server.csr

    ## server.pem
    openssl x509 -req \
                 -days ${days:=7300} \
                 -in ${certs_dir}/server.csr \
                 -CA ${certs_dir}/ca.pem \
                 -CAkey ${certs_dir}/ca-key.pem \
                 -CAcreateserial \
                 -out ${certs_dir}/server.pem \
                 -extfile ${extfile_dir}/server.extfile
    return 0
}

## cert.pem
__do_openssl_gencert_client_for_docker() {
	cat >${extfile_dir}/client.extfile<<-EOF
	keyUsage = critical, digitalSignature
	extendedKeyUsage = clientAuth
	basicConstraints = critical, CA:false
	EOF

    ## cert-key.pem and cert.csr
    openssl req -new \
                -newkey rsa:2048 \
                -days ${days:=7300} \
                -nodes \
                -subj "/O=root.client" \
                -keyout ${certs_dir}/cert-key.pem \
                -out ${certs_dir}/cert.csr

    ## cert.pem
    openssl x509 -req \
                 -days ${days:7300} \
                 -in ${certs_dir}/cert.csr \
                 -CA ${certs_dir}/ca.pem \
                 -CAkey ${certs_dir}/ca-key.pem \
                 -CAcreateserial \
                 -out ${certs_dir}/cert.pem \
                 -extfile ${extfile_dir}/client.extfile
    return 0
}

exit 0
```

## END
