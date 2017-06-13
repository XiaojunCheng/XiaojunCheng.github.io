# 常用Shell命令

## `排序`

```
sort -n -r -k 3
```

`示例`

```
grep file_vss_filter$'\t' mq_2016.10.18.txt | sort -n -r -k 3 | awk 'BEGIN{FS="\t"} {sum+=$3} END {print sum}'
```