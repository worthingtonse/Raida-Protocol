# Key Manager

Key Manager is a CLI program that creates and destroys keys.
The following keys can be managed:

- Admin Key
- User Key
- KYC Key
- Treasurer Key
- RAIDA Key
- Depository
  - Depository Keys

You need an Admin Key in order to create the other key types.
If a RAIDA server is run in the 'Init' mode then the admin key is not required

Init mode can be set in the RAIDA configuration file:
```
init_mode = 1
```

Key Manager can also mint coins. A treasurer key is needed for that purpose.
On top of that any user with KYCADMIN permission can list files and folders for any user on the RAIDA
'listfolder' and 'getfile' commands can be used

The full list of possible arguments:

```
# ./dist/wyoming_key_manager -help
Usage of ./dist/wyoming_key_manager:
./dist/wyoming_key_manager [-debug] <operation> <args>
./dist/wyoming_key_manager [-help]
./dist/wyoming_key_manager [-version]
./dist/wyoming_key_manager createkey <type>
./dist/wyoming_key_manager deletekey <type> <key_id>
./dist/wyoming_key_manager mintcoins <name> <denomination> <count>
./dist/wyoming_key_manager listfolder <user_id> <path>
./dist/wyoming_key_manager getfile <user_id> <path>
<type> Key type. One of: kyc, kycadmin, user, treasurer, raida, admin
<key_id> Key ID (serial number)
<name> Wallet Name
<denomination> Coin Denomination (from -7 to 7)

  -cli
        RUN in CLI mode
  -cliport string
        Listen a specific port (works in CLI mode only)
  -debug
        Display Debug Information
  -help
        Show Usage
  -home string
        Home Directory
  -uipath string
        UI Path
  -version
        Display version

```

Examples:

# Create Admin Key

Error:
```
$ ./dist/wyoming_key_manager createkey admin
{"code":4100,"message":"No encryption coin found","details":null}
```

Success:
```
$ ./dist/wyoming_key_manager createkey admin
{"code":0,"message":"Key created"}
```

# Create User Key
```
$ ./dist/wyoming_key_manager createkey user
{"code":0,"message":"Key created"}
```

# Create Treasurer Key
```
$ ./dist/wyoming_key_manager createkey treasurer
{"code":0,"message":"Key created"}
```

# Delete Treasurer Key
```
$ ./dist/wyoming_key_manager deletekey treasurer
{"code":0,"message":"Key deleted"}
```

# Minting coins

In order to mint a coin you will want to specify the wallet name, denomination and number of coins. 
The denomination range starts from 0.0000001 (lowest) to 10000000 (highest)
Minted coins will be put into the specified wallet.

You should use the denomination codes found [here](https://github.com/worthingtonse/WEST-Protocol/blob/main/B.%20Header%20Format%3A%20Requests.md#denominations)

```
$ ./dist/wyoming_key_manager mintcoins Default 10 12
{"code":0,"message":"Coins Minted"}
```



# Listing folders

You can list folders for any user. Pass <user_id> as the first argument to the 'listfolder' command

```
$ ./dist/wyoming_key_manager listfolder 345 /kyc
{"code":0,"items":[]}
```

# Downloading files

You can download files from the RAIDA. Pass <user_id> as the first argument to the 'listfolder' command

```
$ ./dist/wyoming_key_manager getfile 345 /kyc > ./myfile.bin
```
