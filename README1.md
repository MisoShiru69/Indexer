resim reset
OP1=$(resim new-account)
export PRIV_KEY1=$(echo "$OP1" | sed -nr "s/Private key: ([[:alnum:]_]+)/\1/p")
export PUB_KEY1=$(echo "$OP1" | sed -nr "s/Public key: ([[:alnum:]_]+)/\1/p")
export a1=$(echo "$OP1" | sed -nr "s/Account component address: ([[:alnum:]_]+)/\1/p")

export XRD=030000000000000000000000000000000000000000000000000004

OP2=$(resim new-token-fixed --name OCI 100000000)
echo $OP2
export OCI=$(echo "$OP2" | sed -nr "s/└─ Resource: ([[:alnum:]_]+)/\1/p")

OP3=$(resim new-token-fixed --name DOGE3 1000000000)
echo $OP3
export DOGE3=$(echo "$OP3" | sed -nr "s/└─ Resource: ([[:alnum:]_]+)/\1/p")

PK_OP=$(resim publish ".")
echo $PK_OP
export PACKAGE=$(echo "$PK_OP" | sed -nr "s/Success! New Package: ([[:alnum:]_]+)/\1/p")
echo $PACKAGE

OCI_CP_OP=$(resim call-function $PACKAGE Ociswap new)
export oci_comp=$(echo "$OCI_CP_OP" | sed -nr "s/.* Component: ([[:alnum:]_]+)/\1/p" | sed '1q;d')
echo $oci_comp

ORACLE_CP_OP=$(resim call-function $PACKAGE Oracle new)
export oracle_comp=$(echo "$ORACLE_CP_OP" | sed -nr "s/.* Component: ([[:alnum:]_]+)/\1/p" | sed '1q;d')
echo $oracle_comp

XRD_INDEX=$(resim call-method $oci_comp create_pool $XRD)
export XRD_INDEX_COMP=$(echo "$XRD_INDEX" | sed -nr "s/.* Component: ([[:alnum:]_]+)/\1/p" | sed '1q;d')
OCI_INDEX=$(resim call-method $oci_comp create_pool $OCI)
export OCI_INDEX_COMP=$(echo "$OCI_INDEX" | sed -nr "s/.* Component: ([[:alnum:]_]+)/\1/p" | sed '1q;d')
DOGE3_INDEX=$(resim call-method $oci_comp create_pool $DOGE3)
export DOGE3_INDEX_COMP=$(echo "$DOGE3_INDEX" | sed -nr "s/.* Component: ([[:alnum:]_]+)/\1/p" | sed '1q;d')

resim call-method $oracle_comp set_price $OCI 0.1
resim call-method $oracle_comp set_price $DOGE3 0.001
resim call-method $oracle_comp set_price $XRD 0.06

resim call-method $oci_comp add_liquidity 100000,$XRD 
resim call-method $oci_comp add_liquidity 100000,$OCI 
resim call-method $oci_comp add_liquidity 100000,$DOGE3 

CP1_OP=$(resim call-function $PACKAGE Indexer new)
export comp1=$(echo "$CP1_OP" | sed -nr "s/.* Component: ([[:alnum:]_]+)/\1/p" | sed '1q;d')
echo $comp1

export index_lp=$(echo "$CP1_OP" | sed -nr "s/.* Resource: ([[:alnum:]_]+)/\1/p" | sed '2q;d')
echo $index_lp

resim call-method $comp1 oci_address $oci_comp
resim call-method $comp1 oracle_address $oracle_comp
resim call-method $oci_comp oracle_address $oracle_comp

resim call-method $comp1 create_index_pool $OCI
resim call-method $comp1 create_index_pool $DOGE3 
resim call-method $comp1 create_index_pool $XRD

resim call-method $comp1 deposit 10,$XRD
resim call-method $comp1 deposit 100,$XRD
resim call-method $comp1 deposit 30,$XRD
