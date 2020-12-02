1. 我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

不应该Wrap这个error ，应该使用Sentinel Error ，让其成为dao层的一部分并在dao层打印此error，因为dao层是对数据库层的封装，用来隐藏底层数据库的细节，在底层可以使用任何的数据库。每个数据库都有自己独特的错误，如果Wrap了error。后续更换数据库时上层业务逻辑也需要跟着改错误类型的判断，不然就会造成逻辑错误。

```go
//下面分别以 函数代表各层 如Dao函数就是dao层

func main(){
    err := Controller()
    if err != nil {
        fmt.printf("original error: %T %v",errors.Cause(err),errors.Cause(err))
        fmt.Printf("stack trace: \n%+v\n",err)
    }
}

func Controller()error {
    err := Service()
    if err != nil {
        //在这里也许会判断 errors.Cause(err) == DaoErrNoRecord
        return nil,errors.WithMessage(err,"Sercice error.")
    }
}

func Service()error{
    err := Dao()
    if err != nil {//这里取Wrap
        return nil,errors.Wrap(err,"Dao error")
    }
}
DaoErrNoRecord = errors.New("now record.")
func Dao()error{
    err := DB()
    if err == nil{
        return nil
    }
    if err == DB1ErrNoRows{ //如果是DB2 也许叫DB2ErrNoRows
        return DaoErrNoRecord
    }
}

DB1ErrNoRows = errors.New("no rows.")
func DB() error {
    return DB1ErrNoRows
}
```