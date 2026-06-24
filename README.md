func (suite *OrderMetadataControllerTestSuite) TestUpdate_InvalidMetaId() {
    Convey("Update with invalid metaId should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/abc",
            "ticket=ADO-123&key=mykey&value=myvalue&status=in-use",
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_MissingTicket() {
    Convey("Update without ticket should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            "key=mykey&value=myvalue&status=in-use",
            http.StatusBadRequest)
    })
}
