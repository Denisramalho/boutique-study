FROM golang:1.26 as build

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -v -o productcatalogservice
RUN ls -la /app/productcatalogservice

FROM gcr.io/distroless/static-debian13
COPY --from=build /app/productcatalogservice /
COPY products.json /
CMD [ "/productcatalogservice" ]
