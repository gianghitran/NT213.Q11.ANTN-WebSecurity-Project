# NT213.Q11.ANTN-WebSecurity-Project

## Automated broken object-level authorization attack detection in REST APIs through OpenAPI to colored petri nets transformation

### Installation:
``` bash
python -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```



```
sudo apt install graphviz
```
### Usage

```
cd src/
python3 execute_replay.py <open_api_specification_path> <event_logs_path>
```


### Example
```
$ python3 execute_replay.py examples/OWASP-Juice-Shop-experiment.yaml logs/real_logs_experiment.log

```
