func (suite *OrderMetadataControllerTestSuite) TestDelete_InvalidMetaId() {
    Convey("Delete with invalid metaId should return 400", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/abc",
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_MissingTicket() {
    Convey("Delete without ticket should return 400", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            "", http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_MetadataNotFound() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error { return orm.ErrNoRows })
    defer patch.Reset()

    Convey("Delete when metadata not found should return 404", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            http.StatusNotFound)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_DBFailure() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Delete",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) {
            return 0, orm.ErrArgs
        })
    defer patch.Reset()

    Convey("Delete fails due to DB error should return 500", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            http.StatusInternalServerError)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_Success() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error { return nil })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Delete",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) { return 1, nil })
    defer patch.Reset()

    Convey("Delete succeeds should return 200", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            http.StatusOK)
    })
}
