// Invalid metaId (parse error or zero)
func (suite *OrderMetadataControllerTestSuite) TestUpdate_InvalidMetaId() {
    Convey("Update with invalid metaId should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/abc/meta/99",
            `ticket=ADO-123&key=mykey&value=myvalue&status=in-use`,
            http.StatusBadRequest)
    })
}

// Missing ticket
func (suite *OrderMetadataControllerTestSuite) TestUpdate_MissingTicket() {
    Convey("Update without ticket should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            `key=mykey&value=myvalue&status=in-use`,
            http.StatusBadRequest)
    })
}

// Ops user restriction in production
func (suite *OrderMetadataControllerTestSuite) TestUpdate_ForbiddenInProduction() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    // Order exists and environment is production
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error {
            return nil
        })
    defer patch.Reset()

    // Simulate IsOpsUser returning false
    patch = gomonkey.ApplyFunc(helpers.IsOpsUser, func(_ func(string) interface{}) bool {
        return false
    })
    defer patch.Reset()

    Convey("Update in production by non-ops user should return 403", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            `ticket=ADO-123&key=mykey&value=myvalue&status=in-use`,
            http.StatusForbidden)
    })
}
