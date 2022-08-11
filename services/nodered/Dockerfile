FROM nodered/node-red:3.0.2-16
RUN npm cache verify
RUN for addonnodes in \
node-red-dashboard \
node-red-node-ping \
node-red-contrib-influxdb \
node-red-contrib-boolean-logic \
node-red-contrib-pid \
node-red-contrib-modbus \
node-red-contrib-telnet-client \
; do \
npm install ${addonnodes} ;\
done;