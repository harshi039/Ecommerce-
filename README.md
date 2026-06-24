func (suite *OrderMetadataControllerTestSuite) TestCreate_Success() {
    Convey("Given valid metadata create request", suite.T(), func() {

        suite.OrderMock.On("CreateOrderMetadata",
            mock.Anything,
            mock.Anything,
            mock.Anything,
        ).Return(nil)

        resp := requestUiOrderMetadata(
            suite,
            "POST",
            "/metadata/order/128?ticket=ADO-1&key=mykey&value=myvalue",
        )

        So(resp.Code, ShouldEqual, http.StatusOK)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestUpdate_Success() {
    Convey("Given valid metadata update request", suite.T(), func() {

        suite.OrderMock.On("UpdateOrderMetadata",
            mock.Anything,
            mock.Anything,
            mock.Anything,
        ).Return(nil)

        resp := requestUiOrderMetadata(
            suite,
            "PUT",
            "/metadata/order/128?id=10&ticket=ADO-1",
        )

        So(resp.Code, ShouldEqual, http.StatusOK)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestDelete_Success() {
    Convey("Given valid metadata delete request", suite.T(), func() {

        suite.OrderMock.On("DeleteOrderMetadata",
            mock.Anything,
            uint64(10),
        ).Return(nil)

        resp := requestUiOrderMetadata(
            suite,
            "DELETE",
            "/metadata/order/128?metaId=10&ticket=ADO-1",
        )

        So(resp.Code, ShouldEqual, http.StatusOK)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestCreate_MissingValue_Should400() {

    Convey("Given missing value", suite.T(), func() {

        resp := requestUiOrderMetadata(
            suite,
            "POST",
            "/metadata/order/128?ticket=ADO-1&key=mykey",
        )

        So(resp.Code, ShouldEqual, http.StatusBadRequest)
        So(resp.Body.String(), ShouldContainSubstring, "value is required")
    })
}

func (suite *OrderMetadataControllerTestSuite) TestCreate_InvalidOrderID_Should400() {

    Convey("Given invalid order id", suite.T(), func() {

        resp := requestUiOrderMetadata(
            suite,
            "POST",
            "/metadata/order/abc?ticket=ADO-1&key=mykey&value=myvalue",
        )

        So(resp.Code, ShouldEqual, http.StatusBadRequest)
    })
}

func (suite *OrderMetadataControllerTestSuite) TestCreate_ServiceFailure_Should500() {

    Convey("Given service failure", suite.T(), func() {

        suite.OrderMock.
            On("CreateOrderMetadata",
                mock.Anything,
                mock.Anything,
                mock.Anything).
            Return(errors.New("db error"))

        resp := requestUiOrderMetadata(
            suite,
            "POST",
            "/metadata/order/128?ticket=ADO-1&key=mykey&value=myvalue",
        )

        So(resp.Code, ShouldEqual, http.StatusInternalServerError)
    })
}
