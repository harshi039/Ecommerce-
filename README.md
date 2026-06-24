// Invalid metaId
func (suite *OrderMetadataControllerTestSuite) TestDelete_InvalidMetaId() {
    Convey("Delete with invalid metaId should return 400", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/abc",
            http.StatusBadRequest)
    })
}

// Missing ticket
func (suite *OrderMetadataControllerTestSuite) TestDelete_MissingTicket() {
    Convey("Delete without ticket should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            "", http.StatusBadRequest)
    })
}

// Ops user restriction in production
func (suite *OrderMetadataControllerTestSuite) TestDelete_ForbiddenInProduction() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    // Order exists and environment is production
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error {
            return nil
        })
    defer patch.Reset()

    patch = gomonkey.ApplyFunc(helpers.IsOpsUser, func(_ func(string) interface{}) bool {
        return false
    })
    defer patch.Reset()

    Convey("Delete in production by non-ops user should return 403", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            http.StatusForbidden)
    })
}
