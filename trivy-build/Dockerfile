FROM public.ecr.aws/ubuntu/ubuntu:20.04_stable
RUN apt-get update && apt-get install -y wget net-tools apt-transport-https gnupg \ 
&& wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | apt-key add - \ 
&& echo deb https://aquasecurity.github.io/trivy-repo/deb bionic main | tee -a /etc/apt/sources.list.d/trivy.list \ 
&& apt-get update \ 
&& apt-get install -y trivy
CMD trivy server --listen 0.0.0.0:8080 
