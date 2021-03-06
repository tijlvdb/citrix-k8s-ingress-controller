# Ingress class support

## What is Ingress class?

In a Kubernetes cluster, there might be multiple ingress controllers and you need to have a way to associate a particular ingress resource with an ingress controller.

You can specify the ingress controller that should handle the ingress resource by using the `kubernetes.io/ingress.class` annotation in your ingress resource definition.

## Citrix ingress controller and Ingress classes

Citrix ingress controller supports accepting multiple ingress resources, which have `kuberneters.io/ingress.class` annotation. Each ingress resource can be associated with only one `ingress.class`. However Ingress Controller might need to handle various ingress resources from different classes.

You can associate Ingress Controller with multiple ingress classes using the `--ingress-classes` argument under `spec` section of the YAML file.

If `ingress-classes` is not specified for the Ingress Controller, then it accepts all ingress resources irrespective of the presence of `kubernetes.io/ingress.class` annotation in the ingress object.

If `ingress-classes` is specified, then Ingress Controller accepts only those ingress resources that match the `kubernetes.io/ingress.class` annotation. Ingress resource without the `ingress.class` annotation is not handled by Ingress Controller in the given case.

!!! note "Note"
    Ingress class names are case-insensitive.

## Sample yaml configurations with Ingress classes

Following is the snippet from a sample yaml file to associate `ingress-classes` with the Ingress Controller. This works in both cases where Ingress Controller runs as a standalone pod or runs as sidecar with Citrix ADC CPX. In the given yaml snippet, following ingress classes are associated with the Ingress Controller.

-  `my-custom-class`

-  `Citrix`

```YAML
spec:
    serviceAccountName: cic-k8s-role
    containers:
    - name: cic-k8s-ingress-controller
      image:"quayio/citrix/citrix-k8s-ingress-controller:latest"
      # specify the ingress classes names to be supportedbyIngress Controller in args section.
      # First line should be --ingress-classes, andeverysubsequent line should be
      # the name of allowed ingress class. In the givenexampletwo classes named
      # "citrix" and "my-custom-class" are accepted. Thiswill be case-insensitive.
      args:
        - --ingress-classes
          Citrix
          my-custom-class
```

Following is the snippet from an Ingress yaml file where the Ingress class association is depicted. In the given example, Ingress resource named `web-ingress` is associated with the ingress class `my-custom-class`. If Citrix ingress controller is configured to accept `my-custom-class`, it processes this Ingress resource.

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
     kubernetes.io/ingress.class: "my-custom-class"
```