FROM golang:1.15 as build-env
WORKDIR /go/src/app
ADD hello-app/main.go /go/src/app
ADD hello-app/go.mod /go/src/app
RUN go mod tidy
RUN go build -o /go/bin/app

FROM gcr.io/distroless/base
LABEL org.opencontainers.image.authors="Enrique Tejeda"
COPY --from=build-env /go/bin/app /
CMD ["/app"]
	
