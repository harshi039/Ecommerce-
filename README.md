package s3

import (
	"bytes"
	"context"
	"fmt"
	"time"

	"github.com/aws/aws-sdk-go-v2/service/s3"
)

func CreateBackup(data []byte) error {

	fileName := fmt.Sprintf("backups/backup-%d.json", time.Now().Unix())

	_, err := Client.PutObject(context.TODO(), &s3.PutObjectInput{
		Bucket: &Bucket,
		Key:    &fileName,
		Body:   bytes.NewReader(data),
	})

	if err != nil {
		return err
	}

	fmt.Println("✅ Backup created:", fileName)
	return nil
}



func ListBackups() ([]map[string]interface{}, error) {

	output, err := Client.ListObjectsV2(context.TODO(), &s3.ListObjectsV2Input{
		Bucket: &Bucket,
		Prefix: strPtr("backups/"),
	})

	if err != nil {
		return nil, err
	}

	var backups []map[string]interface{}

	for _, obj := range output.Contents {
		backup := map[string]interface{}{
			"backupId":        *obj.Key,
			"timestamp":       obj.LastModified.Format(time.RFC3339),
			"size":            obj.Size,
			"storageLocation": "s3://" + Bucket + "/" + *obj.Key,
			"status":          "SUCCESS",
		}

		backups = append(backups, backup)
	}

	return backups, nil
}

func strPtr(s string) *string {
	return &s
}



package scheduler

import (
	"log"
	"time"

	"easyshop-backend/s3"
)

func StartBackupScheduler() {

	ticker := time.NewTicker(5 * time.Minute)

	go func() {
		for {
			<-ticker.C

			data := []byte(`{"service":"easyshop","type":"auto-backup"}`)
			err := s3.CreateBackup(data)

			if err != nil {
				log.Println("❌ Backup failed:", err)
			}
		}
	}()
}


