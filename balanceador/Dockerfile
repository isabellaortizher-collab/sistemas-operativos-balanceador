FROM httpd:2.4
RUN sed -i 's/#LoadModule proxy_module/LoadModule proxy_module/' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/#LoadModule proxy_http_module/LoadModule proxy_http_module/' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/#LoadModule proxy_balancer_module/LoadModule proxy_balancer_module/' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/#LoadModule slotmem_shm_module/LoadModule slotmem_shm_module/' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/#LoadModule lbmethod_byrequests_module/LoadModule lbmethod_byrequests_module/' /usr/local/apache2/conf/httpd.conf && \
    sed -i 's/#LoadModule status_module/LoadModule status_module/' /usr/local/apache2/conf/httpd.conf
COPY apache.conf /usr/local/apache2/conf/extra/balanceo.conf
RUN echo "Include conf/extra/balanceo.conf" >> /usr/local/apache2/conf/httpd.conf
