

```ad-info
**Reference resources:**
1. [Video link](https://www.youtube.com/watch?v=zDYL22QNiWk&ab_channel=CoreySchafer)
2. [Official Github page](https://github.com/pypa/pipenv)
```

---

## Installation
```shell
pip install pipenv
```

## Installing depencies using `pipenv`
- To install any package (`requests`, let's say): `pipenv install requests`
![[Pasted image 20230222133557.png]]

- Right after we install `requests`, `pipenv` notices that we do not have a VE for this project yet so it creates one. 
```
Creating a virtualenv for this project...
```

- It also creates a `Pipfile` and `Pipfile.lock` provides us the location.
```
Pipfile: /Users/ayush.nema/Downloads/codes/Pipfile
```

```bash
> tree .   
.
├── Pipfile
├── Pipfile.lock
├── delete_files.py
├── download_files.py
...
```
- The `pipfile` is a file that describes our environment and how we can build it back. This is similar to `requirements.txt` file.

- The python version and location used in creating the VE is also mentioned:
```
Using /Library/Frameworks/Python.framework/Versions/3.9/bin/python3 (3.9.2) to create virtualenv...
```

- The created VE is stored at →
```
Virtualenv location: /Users/ayush.nema/.local/share/virtualenvs/codes-CJK13cag
```
virtual env once created, it proceeds with regular install of `requests` (`Installing requests...`).

- After installation, it proceeds to include `requests` to `Pipfile`
```
Adding requests to Pipfile's [packages] ...
✔ Installation Succeeded
```
if the Pipenv file is not found, it creates one.
```
Pipfile.lock not found, creating...
```

- We can either activate the newly created VE (`pipenv shell`) or use it even without activating (`pipenv run`). 
```
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

## `Pipfile` and `Pipfile.lock`
### > `Pipfile`
```
[[source]]  
url = "https://pypi.org/simple"  
verify_ssl = true  
name = "pypi"  
  
[packages]  
requests = "*"  
  
[dev-packages]  
  
[requires]  
python_version = "3.9"
```
The format this file uses is called TOML. It is considered to the minimum format which contains keys and values within the section.

- `requests = "*"` indicates that we did not specified any version of requests while installation. Otherwise this would have mentioned a version tag here. 
- It also specifies the  `python_version` required. This can be changed (mentioned in later section).

### > `Pipfile.lock`
```
{  
    "_meta": {  
        "hash": {  
            "sha256": "b8c2e1580c53e383cfe4254c1f16560b855d984fde8b2beb3bf6ee8fc2fe5a22"  
        },  
        "pipfile-spec": 6,  
        "requires": {  
            "python_version": "3.9"  
        },  
        "sources": [  
            {  
                "name": "pypi",  
                "url": "https://pypi.org/simple",  
                "verify_ssl": true  
            }  
        ]  
    },  
    "default": {  
        "certifi": {  
            "hashes": [  
                "sha256:35824b4c3a97115964b408844d64aa14db1cc518f6562e8d7261699d1350a9e3",  
"sha256:4ad3232f5e926d6718ec31cfc1fcadfde020920e278684144551c91769c7bc18"  
            ],  
            "markers": "python_version >= '3.6'",  
            "version": "==2022.12.7"  
        },  
        "charset-normalizer": {  
            "hashes": [  
"sha256:00d3ffdaafe92a5dc603cb9bd5111aaa36dfa187c8285c543be562e61b755f6b",  
"sha256:024e606be3ed92216e2b6952ed859d86b4cfa52cd5bc5f050e7dc28f9b43ec42",    
            ],  
            "version": "==3.0.1"  
        },  
        "idna": {  
            "hashes": [  
                "sha256:814f528e8dead7d329833b91c5faa87d60bf71824cd12a7530b5526063d02cb4",  
"sha256:90b77e79eaa3eba6de819a0c442c0b4ceefc341a7a2ab77d7562bf49f425c5c2"  
            ],  
            "markers": "python_version >= '3.5'",  
            "version": "==3.4"  
        },  
        "requests": {  
            "hashes": [  
                "sha256:64299f4909223da747622c030b781c0d7811e359c37124b4bd368fb8c6518baa",  
"sha256:98b1b2782e3c6c4904938b84c0eb932721069dfdb9134313beff7c83c2df24bf"  
            ],  
            "index": "pypi",  
            "version": "==2.28.2"  
        },  
        "urllib3": {  
            "hashes": [  
                "sha256:076907bf8fd355cde77728471316625a4d2f7e713c125f51953bb5b3eecf4f72",  
"sha256:75edcdc2f7d85b137124a6c3c9fc3933cdeaa12ecb9a6a959f22797a0feca7e1"  
            ],  
            "markers": "python_version >= '2.7' and python_version not in '3.0, 3.1, 3.2, 3.3, 3.4, 3.5'",  
            "version": "==1.26.14"  
        }  
    },  
    "develop": {}  
}
```
- this file is not meant to be changed manually.
- has more detailed information about the current environment.
- also specifies the sub-dependencies of the installed library. 
	- each of those sub-dep is mentioned here with exact version info.
- helps in getting more deterministic build of VE


## Usage
##### **1. Activating virtual environment created by `pipenv` →**
```shell
$ pipenv shell 
```

![[Pasted image 20230222142318.png]]

- Checking which python is used for the interpreter within VE
	- see the o/p of `sys.executable`
	- in the python window above, if we now try to `import requests` it works without any errors!
	- more info for path setting, follow the [link](https://www.youtube.com/watch?v=PUIE7CPANfo&ab_channel=CoreySchafer)
![[Pasted image 20230222142512.png]]


##### **2. Deactivate the virtual env→**
Just type `exit` and that's it.
```shell
$ exit
```
(`deactivate` for exiting usual VE, and `conda deactivate` for exiting conda env)


##### **3. Running the commands within VE w/o explicitly activating →**
```shell
$ pipenv run python
```

Launching python again to demonstrate this:
![[Pasted image 20230222143621.png]]
(notice the executable still uses the same python from the VE and never actually activated the VE this no need to run `exit` again)


##### **4. Executing scripts →**
```shell
$ pipenv run python script.py
```


##### **5. Installing multiple packages →**
One way is to repetetiely use `pipenv install <package_name>` command. Other way is to install via requirements file itself.
```shell
$ pipenv install -r ../dir_name/requirements.txt
```

```
> pipenv install -r requirements.txt 
Requirements file provided! Importing into Pipfile...
Pipfile.lock (fe5a22) out of date, updating to (6f4904)...
Locking [packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success!
Locking [dev-packages] dependencies...
Updated Pipfile.lock (f8b4ed96b67340f1bafae30a206c3eb6c020c98c03cd3395dbebaf6caf6f4904)!
Installing dependencies from Pipfile.lock (6f4904)...
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

Open the `Pipfile` and `Pipfile.lock` to check the list of new dependencies installed (as mentioned in requirements file).

Similar to `pip freeze` where we can see all the installed package, pipenv has an alternative.
```shell
$ pipenv lock -r
```
the output can also be shipped as requirements file as:
```shell
$ pipenv lock -r > requirements.txt
```

##### **6. Installing packages only for test (dev) env →**
```shell
$ pipenv install pytest --dev
```
The installation looks routine but it make changes in `Pipfile`.

This appears while installation.
```
Adding pytest to Pipfile's [dev-packages] ...
```

```
[[source]]  
url = "https://pypi.org/simple"  
verify_ssl = true  
name = "pypi"  
  
[packages]  
requests = "==2.28.2"  
boto3 = "==1.26.72"  
botocore = "==1.29.72"  
ffmpeg-python = "==0.2.0"  
huggingface-hub = "==0.12.1"  
joblib = "==1.2.0"  
librosa = "==0.9.2"  
loguru = "==0.6.0"  
matplotlib = "==3.7.0"  
numpy = "==1.23.5"  
openai-whisper = "==20230124"  
optuna = "==3.1.0"  
pandas = "==1.5.3" 
pipenv = "==2023.2.18"  
pydub = "==0.25.1"  
scikit-learn = "==1.2.1"  
scipy = "==1.10.1"   
tqdm = "==4.64.1"  
transformers = "==4.26.1"  
urllib3 = "==1.26.14"  
virtualenv = "==20.19.0"  
  
[dev-packages]  
pytest = "*"  
  
[requires]  
python_version = "3.9"
```
(notice the `dev-packages` section)

If we are creating new VE and want to install only `dev-packages` then use: 
```shell
$ pipenv install -d pytest
```



##### **7. Uninstalling a package →**
```shell
$ pipenv uninstall requests
```
![[Pasted image 20230222150403.png]]


##### **8. Removing the virtual environment →**
```shell
$ pipenv --rm
```
This only removes the VE but keeps all essential files to reinstall i.e. `Pipfile` and `Pipfile.lock`.


##### **9. Re-installing env based on `Pipfile` & `Pipfile.lock`**
```shell
$ pipenv install
```


##### **10. Viewing path to installed environment**
```shell
$ pipenv --venv
```
Since the VE created by pipenv is similar to the one created by pip, we can still go to above path and activate manually.

```shell
 $ source /Users/ayush.nema/.local/share/virtualenvs/codes-CJK13cag/bin/activate
```


##### **11. Dependency graph**
Identifying which subpackage is marked as dependency to which installed package.
```shell
$ pipenv graph
```

Truncated response example
```
> pipenv graph
aiohttp==3.8.4
  - aiosignal [required: >=1.1.2, installed: 1.3.1]
    - frozenlist [required: >=1.1.0, installed: 1.3.3]
  - async-timeout [required: >=4.0.0a3,<5.0, installed: 4.0.2]
  - attrs [required: >=17.3.0, installed: 22.2.0]
  - charset-normalizer [required: >=2.0,<4.0, installed: 3.0.1]
  - frozenlist [required: >=1.1.1, installed: 1.3.3]
  - multidict [required: >=4.5,<7.0, installed: 6.0.4]
  - yarl [required: >=1.0,<2.0, installed: 1.8.2]
    - idna [required: >=2.0, installed: 3.4]
    - multidict [required: >=4.0, installed: 6.0.4]
boto3==1.26.72
  - botocore [required: >=1.29.72,<1.30.0, installed: 1.29.72]
    - jmespath [required: >=0.7.1,<2.0.0, installed: 1.0.1]
    - python-dateutil [required: >=2.1,<3.0.0, installed: 2.8.2]
      - six [required: >=1.5, installed: 1.16.0]
    - urllib3 [required: >=1.25.4,<1.27, installed: 1.26.14]
  - jmespath [required: >=0.7.1,<2.0.0, installed: 1.0.1]
  - s3transfer [required: >=0.6.0,<0.7.0, installed: 0.6.0]
    - botocore [required: >=1.12.36,<2.0a.0, installed: 1.29.72]
      - jmespath [required: >=0.7.1,<2.0.0, installed: 1.0.1]
      - python-dateutil [required: >=2.1,<3.0.0, installed: 2.8.2]
        - six [required: >=1.5, installed: 1.16.0]
      - urllib3 [required: >=1.25.4,<1.27, installed: 1.26.14]

```



## Advanced usage
##### **(I) Changing python version**
Old way → export the dependencies, delete the VE, and recreate from scratch.

- go to `Pipfile`, change the python version to the required one like "3.9" → "3.8"
- go to terminal and `pipenv --python 3.8`
- to confirm the change in python version:
	- run `pipenv run python`. This shows the updated version
	- `import sys` and `sys.executable` to show the python directory in VE.


##### **(II) Identifying known security vunerabilities for installed packages**
```shell
$ pipenv check
```

```
> pipenv check
Checking PEP 508 requirements...
Passed!
Checking installed packages for vulnerabilities...
+======================================================================================================================================================================================================================================+

                               /$$$$$$            /$$
                              /$$__  $$          | $$
           /$$$$$$$  /$$$$$$ | $$  \__//$$$$$$  /$$$$$$   /$$   /$$
          /$$_____/ |____  $$| $$$$   /$$__  $$|_  $$_/  | $$  | $$
         |  $$$$$$   /$$$$$$$| $$_/  | $$$$$$$$  | $$    | $$  | $$
          \____  $$ /$$__  $$| $$    | $$_____/  | $$ /$$| $$  | $$
          /$$$$$$$/|  $$$$$$$| $$    |  $$$$$$$  |  $$$$/|  $$$$$$$
         |_______/  \_______/|__/     \_______/   \___/   \____  $$
                                                          /$$  | $$
                                                         |  $$$$$$/
  by pyup.io                                              \______/

+======================================================================================================================================================================================================================================+

 REPORT 

  Safety v2.3.2 is scanning for Vulnerabilities...
  Scanning dependencies in your files:

  -> /var/folders/px/w4jww6m12l7f8yn3qm2sfqfrvqhk7l/T/codes-CJK13cagbxopf7l7_requirements.txt

  Found and scanned 142 packages
  Timestamp 2023-02-22 15:24:11
  2 vulnerabilities found
  0 vulnerabilities ignored

+======================================================================================================================================================================================================================================+
 VULNERABILITIES FOUND 
+======================================================================================================================================================================================================================================+

-> Vulnerability found in mpmath version 1.2.1
   Vulnerability ID: 51549
   Affected spec: >=1.0.0,<=1.2.1
   ADVISORY: Mpmath v1.0.0 through v1.2.1 is affected by CVE-2021-29063: Regular Expression Denial of Service (ReDOS) vulnerability when the mpmathify function is called. The flaw has been fixed in the master branch of the
   Github repository.https://github.com/fredrik-johansson/mpmath/commit/46d44c3c8f3244017fe1eb102d564eb4ab8ef750
   CVE-2021-29063
   For more information, please visit https://pyup.io/v/51549/742


-> Vulnerability found in protobuf version 3.20.1
   Vulnerability ID: 51167
   Affected spec: >=3.20.0rc0,<3.20.2
   ADVISORY: Protobuf 3.18.3, 3.19.5, 3.20.2 and 4.21.6 include a fix for CVE-2022-1941: A parsing vulnerability for the MessageSet type in the ProtocolBuffers versions prior to and including 3.16.1, 3.17.3, 3.18.2, 3.19.4,
   3.20.1 and 3.21.5 for protobuf-cpp, and versions prior to and including 3.16.1, 3.17.3, 3.18.2, 3.19.4, 3.20.1 and 4.21.5 for protobuf-python can lead to out of memory failures. A specially crafted message with multiple key-...
   CVE-2022-1941
   For more information, please visit https://pyup.io/v/51167/742

 Scan was completed. 2 vulnerabilities were found. 

+======================================================================================================================================================================================================================================+
   REMEDIATIONS

  2 vulnerabilities were found in 2 packages. For detailed remediation & fix recommendations, upgrade to a commercial license. 

+======================================================================================================================================================================================================================================+
```


##### **(III) Finalizing the dependencies**
Once the code is tested and ready to be moved to production. The requirements can be frozen with exact hash codes and versions. This is exclusively important for manually installed packages where the version was not mentioned while installation like `requests = "*"` (in `Pipfile`), in production when the latest package reinstalls it might crash the app. Thus, this finalizing feature simply records the current version of all dependencies and store in `Pipfile.lock` (thats why we ignore the `Pipfile` and use `Pipfile.lock` for dependency installation in production scenarios).
```shell
$ pipenv lock
```

```
> pipenv lock 
Locking [packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success!
Locking [dev-packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success!
Updated Pipfile.lock (ade38c8e90046cb1e07aa0c25a720808ba5f835ae025b2a23c8ed6a98c6298cf)!
```
Once this is done, we can shop the `Pipfile.lock` to production. And then run:
```shell
$ pipenv install --ignore-pipfile
```
This will create the VE using `Pipfile.lock` but ignore the `Pipfile` which is usually used for creating VE.


##### **(IV) Creating environment variables**
Creating  a `.env` file to store all env vars.
- go to project dir, create a new file `.env` (no prefix)
- enter dummy credentials like `SECRET_KEY="PASS"`
- go to terminal
- load the python to check the env vars got loaded (`$ pipenv run python`)

![[Pasted image 20230222155542.png]]

Note the 2nd line:
```
Loading .env environment variables...
```
And just like this all the env vars can be loaded with ease. Make sure to mention `.env` file in `.gitignore` file. 




## Legend
| short hand | Full form           |
| ---------- | ------------------- |
| VE         | Virtual environment |
|            |                     |