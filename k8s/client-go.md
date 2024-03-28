## ready探测
deployment的声明中,在container字段下，加入readinessProbe：
1. 通过探测某个文件是否存在来控制Ready状态
```golang
"readinessProbe": map[string]interface{}{
    "exec": map[string]interface{}{
        "command": []interface{}{
            "cat",
            "/usr/src/app/ready.txt",
        },
    },
    "failureThreshold":    int64(10),
    "initialDelaySeconds": int64(60),
    "periodSeconds":       int64(10),
    "timeoutSeconds":      5,
},
```
2. 探测某个端口通不通



## PVC挂载

1. 创建PVC， pvcName="storage-claim"
```golang
func (impl *cloudImpl) GetPVC(namespace string, pvcName string) *unstructured.Unstructured {
	pvc := &unstructured.Unstructured{
		Object: map[string]interface{}{
			"apiVersion": "v1",
			"kind":       "PersistentVolumeClaim",
			"metadata": map[string]interface{}{
				"name": pvcName,
			},
			"spec": map[string]interface{}{
				"apiVersion": "v1",
				"kind":       "PersistentVolumeClaim",
				"metadata": map[string]interface{}{
					"name":      pvcName,
					"namespace": namespace,
				},
				"accessModes": []string{"ReadWriteMany"},
				"resources": map[string]interface{}{
					"requests": map[string]interface{}{
						"storage": "1Gi",
					},
				},
				"storageClassName": impl.cfg.Pod.StorageClassName,
			},
		},
	}
	return pvc
}
```

2. 和container平级的路径下声明PVC
```golang
    "volumes": []map[string]interface{}{
            {
                "name": "storage-volume",
                "persistentVolumeClaim": map[string]interface{}{
                    "claimName": "storage-claim",
                },
            },
        },
```

3. container下添加volumeMounts字段
```golang
    "volumeMounts": []map[string]interface{}{
    	{
    		"name":      "storage-volume",
    		"mountPath": "/home/nginx/store",
    	},
    },
```