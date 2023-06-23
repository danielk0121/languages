```shell
aws rds describe-db-snapshots --db-instance-identifier prod-maria-230619 --query '
    reverse(sort_by(DBSnapshots,&SnapshotCreateTime))
    [?contains(DBSnapshotIdentifier, `prod-maria-230619-2023-06-`) == `true`]
    .{
        DBSnapshotIdentifier:DBSnapshotIdentifier,
        DBSnapshotArn:DBSnapshotArn,
        SnapshotCreateTime:SnapshotCreateTime,
        DBInstanceIdentifier:DBInstanceIdentifier,
        SnapshotType:SnapshotType
    }
' --no-cli-pager --output=table
```