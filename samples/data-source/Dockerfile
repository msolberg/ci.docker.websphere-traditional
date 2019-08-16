FROM ibmcom/websphere-traditional
COPY app.ear /work/config/app.ear
COPY install_app.py /work/config/install_app.py
COPY datasource.props /work/config/datasource.props
COPY was-config.props /work/config/was-config.props
RUN /work/configure.sh
