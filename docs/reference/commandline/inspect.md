---
title: "inspect"
description: "The inspect command description and usage"
keywords: "inspect, container, json"
---

# inspect

```markdown
Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker object(s) (e.g. container, image, volume,
network, node, service, or task) identified by name or ID

Options:
  -f, --format       Format the output using the given Go template
      --help         Print usage
  -s, --size         Display total file sizes if the type is container
      --type         Return JSON for specified type
```

## Description

Docker inspect provides detailed information on constructs controlled by Docker.

By default, `docker inspect` will render results in a JSON array.

## Request a custom response format (--format)

If a format is specified, the given template will be executed for each result.

Go's [text/template](http://golang.org/pkg/text/template/) package
describes all the details of the format.

## Specify target type (--type)

`--type container|image|node|network|secret|service|volume|task|plugin`

The `docker inspect` command matches any type of object by either ID or name.
In some cases multiple type of objects (for example, a container and a volume)
exist with the same name, making the result ambiguous.

To restrict `docker inspect` to a specific type of object, use the `--type`
option.

The following example inspects a _volume_ named "myvolume"

```console
$ docker inspect --type=volume myvolume
```

## Examples

### Get an instance's IP address

For the most part, you can pick out any field from the JSON in a fairly
straightforward manner.

```console
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $INSTANCE_ID
```

### Get an instance's MAC address

```console
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.MacAddress}}{{end}}' $INSTANCE_ID
```

### Get an instance's log path

```console
$ docker inspect --format='{{.LogPath}}' $INSTANCE_ID
```

### Get an instance's image name

```console
$ docker inspect --format='{{.Config.Image}}' $INSTANCE_ID
```

### List all port bindings

You can loop over arrays and maps in the results to produce simple text
output:

```console
$ docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' $INSTANCE_ID
```

### Find a specific port mapping

The `.Field` syntax doesn't work when the field name begins with a
number, but the template language's `index` function does. The
`.NetworkSettings.Ports` section contains a map of the internal port
mappings to a list of external address/port objects. To grab just the
numeric public port, you use `index` to find the specific port map, and
then `index` 0 contains the first object inside of that. Then we ask for
the `HostPort` field to get the public address.

```console
$ docker inspect --format='{{(index (index .NetworkSettings.Ports "8787/tcp") 0).HostPort}}' $INSTANCE_ID
```

### Get a subsection in JSON format

If you request a field which is itself a structure containing other
fields, by default you get a Go-style dump of the inner values.
Docker adds a template function, `json`, which can be applied to get
results in JSON format.

```console
$ docker inspect --format='{{json .Config}}' $INSTANCE_ID
```
