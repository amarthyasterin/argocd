#AWS Application Load Balancers 
--------------------------------

Ingress Configuration
----------------------
Argo CD API server runs both a gRPC server (used by the CLI), as well as a HTTP/HTTPS server (used by the UI). Both protocols are exposed by the argocd-server service object on the following ports:

443 - gRPC/HTTPS
80 - HTTP (redirects to HTTPS)
There are several ways how Ingress can be configured.

Ambassador
----------
The Ambassador Edge Stack can be used as a Kubernetes ingress controller with automatic TLS termination and routing capabilities for both the CLI and the UI.

The API server should be run with TLS disabled. Edit the argocd-server deployment to add the --insecure flag to the argocd-server command, or simply set server.insecure: "true" in the argocd-cmd-params-cm ConfigMap as described here. Given the argocd CLI includes the port number in the request host header, 2 Mappings are required.

(ALBs)
------
AWS ALBs can be used as an L7 Load Balancer for both UI and gRPC traffic, whereas Classic ELBs and NLBs can be used as L4 Load Balancers for both.

When using an ALB, you'll want to create a second service for argocd-server. This is necessary because we need to tell the ALB to send the GRPC traffic to a different target group then the UI traffic, since the backend protocol is HTTP2 instead of HTTP1.

Once we create this service, we can configure the Ingress to conditionally route all application/grpc traffic to the new HTTP2 backend, using the alb.ingress.kubernetes.io/conditions annotation, as seen below. Note: The value after the . in the condition annotation must be the same name as the service that you want traffic to route to - and will be applied on any path with a matching serviceName.
