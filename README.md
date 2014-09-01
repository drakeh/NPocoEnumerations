Enumeration Support with NPoco
=================
This is a sample of how to implement support for the Headspring Enumeration class when using NPoco.  
https://github.com/HeadspringLabs/Enumeration

Getting right to it
=================
```C#
public static class DatabaseFactory
{
    private static NPoco.DatabaseFactory InitializeFactory()
    {
        return NPoco.DatabaseFactory.Config(x =>
                {
                    x.WithMapper(new EnumerationMapper());
                });
    }
}
```

```C#
public class EnumerationMapper : DefaultMapper
    {
        public override System.Func<object, object> GetToDbConverter(System.Type destType, System.Type SourceType)
        {
            if (SourceType.IsEnumeration())
                return x => SourceType.GetProperty("Value").GetValue(x, new object[] { });
            return base.GetToDbConverter(destType, SourceType);
        }

        public override System.Func<object, object> GetToDbConverter(System.Type destType, System.Reflection.MemberInfo sourceMemberInfo)
        {
            return GetToDbConverter(destType, sourceMemberInfo.GetMemberInfoType());
        }

        public override System.Func<object, object> GetFromDbConverter(System.Type destType, System.Type sourceType)
        {
            if(destType.IsEnumeration())
                return x => destType.BaseType.GetMethod("FromValue").Invoke(null, new[] { x });
            return base.GetFromDbConverter(destType, sourceType);
        }

        public override System.Func<object, object> GetFromDbConverter(System.Reflection.MemberInfo destMemberInfo, System.Type sourceType)
        {
            return GetFromDbConverter(destMemberInfo.GetMemberInfoType(), sourceType);
        }
    }
```

LINQ Query Support
==================

NPoco 2.0 introduced the ability to [fetch objects using LINQ queries](https://github.com/schotime/NPoco/wiki/Simple-linq-queries). This sample code also demonstrates how to override NPoco's default parameter converter to allow the Headspring Enumeration to be used in a LINQ query to fetch database objects. A sample usage follows: 

```C#
IDatabase db = new Database("connString");
db.FetchWhere<BlogPost>(x => x.Category == Category.Books);
```

Sources
=================
Thanks to a helpful breakdown here - https://github.com/schotime/NPoco/issues/58