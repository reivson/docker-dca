FROM golang:1.12.4

WORKDIR /helloworld

COPY helloworld.go .

RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .

CMD ["./helloworld"]