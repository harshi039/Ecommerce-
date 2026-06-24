func (suite *OrderMetadataControllerTestSuite) TestDelete_InvalidMetaId() {
    Convey("Delete with invalid metaId should return 400", suite.T(), func() {
        // metaId = "abc" → strconv.Atoi fails
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/abc",
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_MissingTicket() {
    Convey("Delete without ticket should return 400", suite.T(), func() {
        // ticket missing → controller returns 400
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            "", http.StatusBadRequest)
    })
}
