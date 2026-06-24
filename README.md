func (suite *OrderMetadataControllerTestSuite) TestUpdate_InvalidMetaId() {
    Convey("Update with invalid metaId should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/abc/meta/99",
            `ticket=ADO-123&key=mykey&value=myvalue&status=in-use`,
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_MissingTicket() {
    Convey("Update without ticket should return 400", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            `key=mykey&value=myvalue&status=in-use`,
            http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_MetadataNotFound() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error {
            return orm.ErrNoRows
        })
    defer patch.Reset()

    Convey("Update when metadata not found should return 404", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            `ticket=ADO-123&key=mykey&value=myvalue&status=in-use`,
            http.StatusNotFound)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_DBFailure() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Update",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) {
            return 0, orm.ErrArgs
        })
    defer patch.Reset()

    Convey("Update fails due to DB error should return 500", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            `ticket=ADO-123&key=mykey&value=myvalue&status=in-use`,
            http.StatusInternalServerError)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_Success() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error { return nil })
    defer patch.Reset()

    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Update",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) { return 1, nil })
    defer patch.Reset()

    Convey("Update succeeds should return 200", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            `ticket=ADO-123&key=mykey&value=myvalue&status=in-use`,
            http.StatusOK)
    })
}
