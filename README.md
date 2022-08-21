resim reset
OP1=$(resim new-account)
export PRIV_KEY1=$(echo "$OP1" | sed -nr "s/Private key: ([[:alnum:]_]+)/\1/p")
export PUB_KEY1=$(echo "$OP1" | sed -nr "s/Public key: ([[:alnum:]_]+)/\1/p")
export a1=$(echo "$OP1" | sed -nr "s/Account component address: ([[:alnum:]_]+)/\1/p")

export XRD=030000000000000000000000000000000000000000000000000004

OP2=$(resim new-token-fixed --name OCI 10000)
echo $OP2
export OCI=$(echo "$OP2" | sed -nr "s/└─ Resource: ([[:alnum:]_]+)/\1/p")

OP3=$(resim new-token-fixed --name DOGE3 100000)
echo $OP3
export DOGE3=$(echo "$OP3" | sed -nr "s/└─ Resource: ([[:alnum:]_]+)/\1/p")

PK_OP=$(resim publish ".")
echo $PK_OP
export PACKAGE=$(echo "$PK_OP" | sed -nr "s/Success! New Package: ([[:alnum:]_]+)/\1/p")
echo $PACKAGE

CP1_OP=$(resim call-function $PACKAGE Indexer new $OCI $DOGE3)
export comp1=$(echo "$CP1_OP" | sed -nr "s/├─ Component: ([[:alnum:]_]+)/\1/p")
echo $comp1

export comp2=$(echo "$CP1_OP" | sed -nr "s/└─ Component: ([[:alnum:]_]+)/\1/p")
echo $comp2

resim call-method $comp2 create_index_pool $OCI OCI
resim call-method $comp2 create_index_pool $DOGE3 DOGE3
resim call-method $comp2 create_index_pool $XRD XRD
