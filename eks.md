- By default when you create a service of type load balancer in EKS, it will provision a classic load balancer.
- When you install AWS Load balancer controller, that default has been changed to NLB.
- For ingress, AWS Load balancer controller will create ALB.
- AWS Load balancer controller in this case is an example of Ingress controller.
- By mentioning IngressClassName, you're telling which ingress controller to be used to provision Load balancers.

  *IPV6*:
  - The address range of IPv6 is 4 billion times the range of IPV4.
