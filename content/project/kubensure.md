---
title: kubensure
date: 2022-02-19T07:00:13+00:00
img: "/images/projects/kubensure.webp"
description: CLI tool implemented in Golang to ensure network communications between
  Kubernetes pods
github: https://github.com/PhilRanzato/kubensure

---
!\[Build\]([https://github.com/PhilRanzato/kubensure/workflows/Go/badge.svg](https://github.com/PhilRanzato/kubensure/workflows/Go/badge.svg "https://github.com/PhilRanzato/kubensure/workflows/Go/badge.svg"))

Ensure consistency in Kubernetes cluster's resources

## Application

This application is meant to verify communication among Kubernetes pods. It consists in a Golang Command line interface powered by [Cobra](https://github.com/spf13/cobra/cobra "Cobra") 

### How to use

    $ kubensure
    kubensure is a CLI tool that allows kubernetes cluster admins to check their policies are respected in the cluster.
    
    Usage:
      kubensure [command]
    
    Available Commands:
      connection  Check connection from a pod to a service or to another pod or to an external endpoint.
      help        Help about any command
    
    Flags:
          --config string   config file (default is $HOME/.kubensure.yaml)
      -h, --help            help for kubensure
      -t, --toggle          Help message for toggle
    
    Use "kubensure [command] --help" for more information about a command.

    $ kubensure connection --help
    
    Check connection from a pod to a service or to another pod or to an external endpoint.
    
    Usage:
      kubensure connection [command]
    
    Available Commands:
      pod-to-ext  Check connection from a pod to an external endpoint.
      pod-to-pod  Check connection from a pod to another pod.
      pod-to-svc  Check connection from a pod to a service.
    
    Flags:
      -h, --help   help for connection
    
    Global Flags:
          --config string   config file (default is $HOME/.kubensure.yaml)
    
    Use "kubensure connection [command] --help" for more information about a command.

    $ kubensure connection pod-to-ext --help
    
    Check connection from a pod to an external endpoint.
    
    Usage examples:
    
      # Ensure pod 'example' of namespace 'test' can connect to https://kubernetes.io
    
      kubensure connection pod-to-ext example -n test https://kubernetes.io
    
      # Ensure pod 'example' of namespace 'test' can connect to http://192.168.100.112:90
    
      kubensure connection pod-to-ext example -n test http://192.168.100.112 --ext-port 90
    
    Usage:
      kubensure connection pod-to-ext [flags]
    
    Flags:
      -p, --ext-port int    External endpoint port (default 443)
      -h, --help            help for pod-to-ext
      -n, --pod-ns string   Pod namespace (default "default")
    
    Global Flags:
          --config string   config file (default is $HOME/.kubensure.yaml)

    $ kubensure connection pod-to-pod --help
    
    Check connection from a pod to another pod.
    
    Usage examples:
    
      # Ensure pod 'example' of namespace 'test' can connect to pod 'target' in namespace 'pod-test'
    
      kubensure connection pod-to-pod example -n test target -t pod-test
    
      # Ensure pod 'example' of namespace 'test' can connect to pod 'target' in namespace 'pod-test' on port 8000
    
      kubensure connection pod-to-pod example -n test target -t pod-test -p 8000
    
    Usage:
      kubensure connection pod-to-pod [flags]
    
    Flags:
      -h, --help               help for pod-to-pod
      -n, --pod-ns string      Pod namespace (default "default")
      -t, --target-ns string   Target Pod namespace (default "default")
      -p, --target-port int    Target Pod port
    
    Global Flags:
          --config string   config file (default is $HOME/.kubensure.yaml)

    $ kubensure connection pod-to-svc --help
    
    Check connection from a pod to a service.
    
    Usage examples:
    
      # Ensure pod 'example' of namespace 'test' can connect to service 'svc-example' in namespace 'svc-test'
    
      kubensure connection pod-to-svc example -n test svc-example -s svc-test
    
    Usage:
      kubensure connection pod-to-svc [flags]
    
    Flags:
      -h, --help            help for pod-to-svc
      -n, --pod-ns string   Pod namespace (default "default")
      -t, --svc-ns string   Target Service namespace (default "default")
      -p, --svc-port int    Target Service port
    
    Global Flags:
          --config string   config file (default is $HOME/.kubensure.yaml)

### How to install

Download the executable from the release section and place it under your PATH