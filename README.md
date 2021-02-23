# Bootstrapping Scripts for Docker Images

Set environment variables to create files, manipulate JSON and run scripts
to bootstrap your container without rebuilding your image.

## Usage

```docker
FROM icetan/runr as runr

FROM alpine

RUN apk add --no-cache bash jq

COPY --from=runr /runr /bin/runr

ENTRYPOINT [ "/bin/runr" ]
```

## Create Files

```
<ID>           To use as id for file

FILR_OUT_<ID>  Path to write file to
FILR_IN_<ID>   String to put in file
FILR_MOD_<ID>  Set file chmod
FILR_OWN_<ID>  Set owner of file
```

E.g. create a simple text file with read-write access for `some_user` and
read-only access for `some_group`:

```sh
FILR_OUT_config=/root/config
FILR_IN_config="This will be written to the file /root/config"
FILR_MOD_config=0600
FILR_OWN_config=some_user:some_group
```

## Manipulate JSON Files

```
<ID>                  To use as id for manipulations
<PROP>                A JSON property/path, sub-properties are delimited by _
                      and arrays are iterated with __
<VAR>                 jq variable name

JSNR_OUT_<ID>         Path to write JSON file to
JSNR_IN_<ID>          Path to JSON file to use as input (defaults to value of JSNR_OUT_<ID>)
JSNR_<ID>_SET         JSON value to use as input (overrides JSNR_IN_<ID>)
JSNR_<ID>_JQ          jq script to execute on JSON input (default: .)
JSNR_<ID>_ARG_<VAR>   JSON value to give jq as argument with variable name VAR
JSNR_<ID>_SET_<PROP>  JSON value to set PROP to
JSNR_<ID>_STR_<PROP>  String to set PROP to
JSNR_<ID>_MUL_<PROP>  JSON value to multiply PROP with
JSNR_<ID>_ADD_<PROP>  JSON value to add PROP with
```

E.g. update the `a` and `b` properties of `/root/config.json`:

```sh
JSNR_OUT_config=/root/config.json
JSNR_config_SET_a='["a","array"]'
JSNR_config_STR_b="A string value"
```

## Pre-run Scripts

```
<PHASE>       To use as id for script, also determines order of execution

SCTR_<PHASE>  Literal Bash script to execute before main process
```

E.g. run a init script before main process:

```sh
SCTR_000='[[ -f /root/init ]] || echo "I have inited" > /root/init'
```

## Re-map Environment Variables

```
<FROM>       An environment variable name to map another variable from

MAPR_<FROM>  An environment variable name to map <FROM> variable to
```

E.g. create a short alias for a environment variable:

```sh
MAPR_SHORT_VAR=LONG_VARIABLE
```

## Build Image

```sh
docker build -t icetan/runr .
```
