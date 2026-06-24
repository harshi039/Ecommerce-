func (suite *OrderMetadataControllerTestSuite) TestCreate_AuditLogFailureNonFatal() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    // Simulate order exists
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error { return nil })
    defer patch.Reset()

    // Simulate insert succeeds
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Insert",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) { return 42, nil })
    defer patch.Reset()

    // Audit log fails but is non-fatal
    suite.AuditLogMock.On("AddAuditLog", mock.Anything).Return(int64(0), orm.ErrArgs)

    Convey("Create succeeds even if audit log fails", suite.T(), func() {
        requestUiOrderMetadataWithBodyAndCheck(suite, http.MethodPost,
            "/metadata/order/128/create",
            "ticket=ADO-123&key=mykey&value=myvalue",
            http.StatusOK)
    })
}


func (suite *OrderMetadataControllerTestSuite) TestUpdate_AuditLogFailureNonFatal() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    // Metadata exists
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error { return nil })
    defer patch.Reset()

    // Update succeeds
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Update",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) { return 1, nil })
    defer patch.Reset()

    // Audit log fails but controller should still return 200
    suite.AuditLogMock.On("AddAuditLog", mock.Anything).Return(int64(0), orm.ErrArgs)

    Convey("Update succeeds even if audit log fails", suite.T(), func() {
        requestUpdateOrderMetadataWithBodyAndCheck(suite, http.MethodPut,
            "/metadata/order/128/meta/99",
            "ticket=ADO-123&key=mykey&value=myvalue&status=in-use",
            http.StatusOK)
    })
}


func (suite *OrderMetadataControllerTestSuite) TestDelete_AuditLogFailureNonFatal() {
    ormMock := &mocks.Ormer{}
    patch := gomonkey.ApplyFunc(orm.NewOrm, func() orm.Ormer { return ormMock })
    defer patch.Reset()

    // Metadata exists
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Read",
        func(*mocks.Ormer, interface{}, ...string) error { return nil })
    defer patch.Reset()

    // Delete succeeds
    patch = gomonkey.ApplyMethod(reflect.TypeOf(ormMock), "Delete",
        func(*mocks.Ormer, interface{}, ...string) (int64, error) { return 1, nil })
    defer patch.Reset()

    // Audit log fails but controller should still return 200
    suite.AuditLogMock.On("AddAuditLog", mock.Anything).Return(int64(0), orm.ErrArgs)

    Convey("Delete succeeds even if audit log fails", suite.T(), func() {
        requestUiOrderMetadataAndCheck(suite, http.MethodDelete,
            "/metadata/order/128/meta/99",
            http.StatusOK)
    })
}
