# MULTI-STAGE BUILD for IBM Spectrum Scale CSI Driver

FROM golang:1.13.1 AS builder
WORKDIR /go/src/github.com/IBM/ibm-spectrum-scale-csi/driver/
COPY . .
ARG GOFLAGS
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o _output/ibm-spectrum-scale-csi ./cmd/ibm-spectrum-scale-csi
RUN chmod +x _output/ibm-spectrum-scale-csi

FROM registry.access.redhat.com/ubi8-minimal:latest
LABEL name="IBM Spectrum Scale CSI driver" \
      vendor="ibm" \
      version="1.0.1" \
      release="1" \
      run='docker run ibm-spectrum-scale-csi-driver' \
      summary="An implementation of CSI Plugin for the IBM Spectrum Scale product."\
      description="CSI Plugin for IBM Spectrum Scale"\
      maintainers="IBM Spectrum Scale"
COPY licenses /licenses
COPY --from=builder /go/src/github.com/IBM/ibm-spectrum-scale-csi/driver/_output/ibm-spectrum-scale-csi /ibm-spectrum-scale-csi
ENTRYPOINT ["/ibm-spectrum-scale-csi"]
