FROM golang:1.12.4 AS compiler
WORKDIR /helloworld
COPY helloworld.go .
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .

FROM alpine:3.9.3
WORKDIR /root
COPY --from=compiler /helloworld/helloworld .
CMD ["./helloworld"]