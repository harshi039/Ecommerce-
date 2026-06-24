func (suite *OrderMetadataControllerTestSuite) TestUpdate_Success() {
    Convey("Update with valid ticket and key should return 200", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            "ticket=ADO-123&key=mykey&value=myvalue&status=in-use",
            http.StatusOK)
    })
}


func (suite *OrderMetadataControllerTestSuite) TestDelete_Success() {
    Convey("Delete with valid ticket should return 200", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            "ticket=ADO-123", http.StatusOK)
    })
}
