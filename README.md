package s3

import (
	"bytes"
	"context"
	"fmt"
	"mime/multipart"
	"os"

	"github.com/aws/aws-sdk-go-v2/config"
	awss3 "github.com/aws/aws-sdk-go-v2/service/s3"
)

/*
   1️⃣ Interface — THIS is what makes testing possible
*/
type S3Client interface {
	PutObject(
		ctx context.Context,
		params *awss3.PutObjectInput,
		optFns ...func(*awss3.Options),
	) (*awss3.PutObjectOutput, error)

	ListObjectsV2(
		ctx context.Context,
		params *awss3.ListObjectsV2Input,
		optFns ...func(*awss3.Options),
	) (*awss3.ListObjectsV2Output, error)
}

/*
   2️⃣ Globals used by app & tests
*/
var Client S3Client
var Bucket string

/*
   3️⃣ Init S3 (used in real app, NOT in tests)
*/
func InitS3() {
	Bucket = os.Getenv("S3_BUCKET")

	cfg, err := config.LoadDefaultConfig(context.TODO())
	if err != nil {
		panic("Failed to load AWS config: " + err.Error())
	}

	Client = awss3.NewFromConfig(cfg)
	fmt.Println("✅ AWS S3 connected!")
}

/*
   4️⃣ Upload file to S3
*/
func UploadFile(file multipart.File, fileName string) (string, error) {
	buf := new(bytes.Buffer)

	_, err := buf.ReadFrom(file)
	if err != nil {
		return "", err
	}

	_, err = Client.PutObject(context.TODO(), &awss3.PutObjectInput{
		Bucket: &Bucket,
		Key:    &fileName,
		Body:   bytes.NewReader(buf.Bytes()),
	})
	if err != nil {
		return "", err
	}

	url := "https://" + Bucket + ".s3.amazonaws.com/" + fileName
	return url, nil
}
