func (suite *OrderMetadataControllerTestSuite) TestCreate_BadRequest_NoBody() {
    Convey("Create with no body should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodPost,
            "/metadata/order/128/create",
            "", http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestCreate_BadRequest_EmptyParams() {
    Convey("Create with empty params should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodPost,
            "/metadata/order/128/create",
            "ticket=&key=&value=", http.StatusBadRequest)
    })
}


func (suite *OrderMetadataControllerTestSuite) TestUpdate_BadRequest_NoBody() {
    Convey("Update with no body should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            "", http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_BadRequest_EmptyTicket() {
    Convey("Update with empty ticket should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            "ticket=&key=mykey&value=myvalue&status=in-use",
            http.StatusBadRequest)
    })
}


func (suite *OrderMetadataControllerTestSuite) TestDelete_BadRequest_NoTicket() {
    Convey("Delete without ticket should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            "", http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_BadRequest_InvalidMetaId() {
    Convey("Delete with invalid metaId should return 400", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/abc",
            http.StatusBadRequest)
    })
}
