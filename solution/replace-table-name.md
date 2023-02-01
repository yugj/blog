# SQL表名替换

## 方案一 jsqlparser
```xml
  <dependency>
    <groupId>com.github.jsqlparser</groupId>
    <artifactId>jsqlparser</artifactId>
    <version>4.4</version>
  </dependency>
```

```java
import java.lang.reflect.Field;
import java.util.List;
import lombok.Data;
import net.sf.jsqlparser.JSQLParserException;
import net.sf.jsqlparser.parser.CCJSqlParserUtil;
import net.sf.jsqlparser.schema.Table;
import net.sf.jsqlparser.statement.Statement;
import net.sf.jsqlparser.util.TablesNamesFinder;
import org.springframework.util.ReflectionUtils;

/**
 * @author yugj
 * @create 2023/02/01 9:23 AM
 */
@Data
public class TablesNamesParser extends TablesNamesFinder {

  public static void main(String[] args) throws JSQLParserException {
    String originalSql = "select * from abc t1 left join\n"
        + "bbc t2 on t1.aa = t2.bb where abc.content = '\n\tabcdefg'";

    TablesNamesParser tablesNamesParser = new TablesNamesParser();
    Statement statement = CCJSqlParserUtil.parse(originalSql);

    List<String> tableList = tablesNamesParser.getTableList(statement);

    System.out.println(tableList);
    System.out.println(statement);
  }

  @Override
  public void visit(Table table) {

    List<String> otherItemNames = getFieldValue(this, "otherItemNames", List.class);
    List<String> tables = getFieldValue(this, "tables", List.class);

    if (otherItemNames == null || tables == null) {
      return;
    }

    String tableWholeName = extractTableName(table);
    if (!otherItemNames.contains(tableWholeName.toLowerCase()) && !tables.contains(tableWholeName)) {

      String replacedName = tableWholeName + "_version";

      tables.add(replacedName);
      table.setName(replacedName);
    }
  }

  private static <T> T getFieldValue(Object target, String fieldName, Class<T> type) {
    Field field = target instanceof Class
        ? ReflectionUtils.findField((Class<?>) target, fieldName)
        : ReflectionUtils.findField(target.getClass(), fieldName);
    field.setAccessible(true);
    return (T) ReflectionUtils.getField(field, target);
  }
}
```

## 方案2 druid

```xml
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.22</version>
    </dependency>
```

```java

import com.alibaba.druid.sql.SQLUtils;
import com.alibaba.druid.sql.ast.SQLStatement;
import com.alibaba.druid.sql.ast.statement.SQLExprTableSource;
import com.alibaba.druid.sql.dialect.mysql.visitor.MySqlASTVisitorAdapter;
import com.alibaba.druid.util.JdbcConstants;

/**
 * druid impl
 */
public class SimpleTableReplacer5 extends MySqlASTVisitorAdapter {

  private final static String TABLE_SUFFIX = "_version";

  public static void main(String[] args) {

    String originalSql =
        "select * from abc t1 left join\n" + "bbc t2 on t1.aa = t2.bb where abc.content = '\n\tabcdefg'";

    SimpleTableReplacer5 prefixVisitor = new SimpleTableReplacer5();
    SQLStatement statement = SQLUtils.parseSingleStatement(originalSql, JdbcConstants.MYSQL);
    statement.accept(prefixVisitor);
    String convertedSql = SQLUtils.toSQLString(statement, JdbcConstants.MYSQL);

    System.out.println(convertedSql);
  }

  @Override
  public boolean visit(SQLExprTableSource tableSource) {
    String table = tableSource.getExpr().toString();
    if (table.contains("`")) {
      table = table.replace("`", "");
    }

    tableSource.setExpr(table + TABLE_SUFFIX);
    return true;
  }
}
```