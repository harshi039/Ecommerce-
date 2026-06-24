func (suite *OrderMetadataControllerTestSuite) TestCreate_InvalidOrderId() {
    Convey("Create with invalid orderId should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodPost,
            "/metadata/order/abc/create",
            "ticket=ADO-123&key=mykey&value=myvalue",
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestCreate_MissingTicket() {
    Convey("Create without ticket should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodPost,
            "/metadata/order/128/create",
            "key=mykey&value=myvalue",
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestCreate_MissingKey() {
    Convey("Create without key should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodPost,
            "/metadata/order/128/create",
            "ticket=ADO-123&value=myvalue",
            http.StatusBadRequest)
    })
}

