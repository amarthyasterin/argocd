#AWS Application Load Balancers (ALBs) And Classic ELB (HTTP Mode)
--------------------------------------------------------------------
AWS ALBs can be used as an L7 Load Balancer for both UI and gRPC traffic, whereas Classic ELBs and NLBs can be used as L4 Load Balancers for both.

When using an ALB, you'll want to create a second service for argocd-server. This is necessary because we need to tell the ALB to send the GRPC traffic to a different target group then the UI traffic, since the backend protocol is HTTP2 instead of HTTP1.

Once we create this service, we can configure the Ingress to conditionally route all application/grpc traffic to the new HTTP2 backend, using the alb.ingress.kubernetes.io/conditions annotation, as seen below. Note: The value after the . in the condition annotation must be the same name as the service that you want traffic to route to - and will be applied on any path with a matching serviceName.
