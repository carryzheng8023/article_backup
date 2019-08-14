### Hive用户自定义函数(User-Defined Function, UDF)

#### 一、自定义函数类别

##### 1. UDF(User-Defined-Function) ：一进一出；

##### 2. UDAF(User-Defined Aggregation Function)：多进一出，如：count()、max()、min()；

##### 3. UDTF(User-Defined Table-Generating Function) ：一进多出，如：explore()



#### 二、代码示例

[源代码链接](http://svn.apache.org/repos/asf/hive/branches/branch-0.8/ql/src/java/org/apache/hadoop/hive/ql/udf/generic/)


##### 1. 求平均值(UDAF)

```java
package xin.carryzheng.hive.udaf.example;

import java.util.ArrayList;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.parse.SemanticException;
import org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator;
import org.apache.hadoop.hive.serde2.io.DoubleWritable;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.PrimitiveObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.StructField;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.DoubleObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.LongObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorUtils;
import org.apache.hadoop.hive.serde2.typeinfo.PrimitiveTypeInfo;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfo;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.util.StringUtils;

/**
 * GenericUDAFAverage.
 */
@Description(name = "avg", value = "_FUNC_(x) - Returns the mean of a set of numbers")
public class GenericUDAFAverage extends AbstractGenericUDAFResolver {

    static final Log LOG = LogFactory.getLog(GenericUDAFAverage.class.getName());

    //读入参数类型校验，满足条件时返回聚合函数数据处理对象
    @Override
    public GenericUDAFEvaluator getEvaluator(TypeInfo[] parameters)
            throws SemanticException {
        if (parameters.length != 1) {
            throw new UDFArgumentTypeException(parameters.length - 1,
                    "Exactly one argument is expected.");
        }

        if (parameters[0].getCategory() != ObjectInspector.Category.PRIMITIVE) {
            throw new UDFArgumentTypeException(0,
                    "Only primitive type arguments are accepted but "
                            + parameters[0].getTypeName() + " is passed.");
        }
        switch (((PrimitiveTypeInfo) parameters[0]).getPrimitiveCategory()) {
            case BYTE:
            case SHORT:
            case INT:
            case LONG:
            case FLOAT:
            case DOUBLE:
            case STRING:
            case TIMESTAMP:
                return new GenericUDAFAverageEvaluator();
            case BOOLEAN:
            default:
                throw new UDFArgumentTypeException(0,
                        "Only numeric or string type arguments are accepted but "
                                + parameters[0].getTypeName() + " is passed.");
        }
    }

    /**
     * GenericUDAFAverageEvaluator.
     * 自定义静态内部类：数据处理类，继承GenericUDAFEvaluator抽象类
     */
    public static class GenericUDAFAverageEvaluator extends GenericUDAFEvaluator {

        //1.1.定义全局输入输出数据的类型OI实例，用于解析输入输出数据
        // input For PARTIAL1 and COMPLETE
        PrimitiveObjectInspector inputOI;

        // input For PARTIAL2 and FINAL
        // output For PARTIAL1 and PARTIAL2
        StructObjectInspector soi;
        StructField countField;
        StructField sumField;
        LongObjectInspector countFieldOI;
        DoubleObjectInspector sumFieldOI;

        //1.2.定义全局输出数据的类型，用于存储实际数据
        // output For PARTIAL1 and PARTIAL2
        Object[] partialResult;

        // output For FINAL and COMPLETE
        DoubleWritable result;

        /**
         * 初始化：对各个模式处理过程，提取输入数据类型OI，返回输出数据类型OI
         * .每个模式（Mode）都会执行初始化
         * 1.输入参数parameters：
         * .1.1.对于PARTIAL1 和COMPLETE模式来说，是原始数据（单值）
         *    .设定了iterate()方法的输入参数的类型OI为：
         *    .		 PrimitiveObjectInspector 的实现类 WritableDoubleObjectInspector 的实例
         *    .		 通过输入OI实例解析输入参数值
         * .1.2.对于PARTIAL2 和FINAL模式来说，是模式聚合数据（双值）
         *    .设定了merge()方法的输入参数的类型OI为：
         *    .		 StructObjectInspector 的实现类 StandardStructObjectInspector 的实例
         *    .		 通过输入OI实例解析输入参数值
         * 2.返回值OI：
         * .2.1.对于PARTIAL1 和PARTIAL2模式来说，是设定了方法terminatePartial()返回值的OI实例
         *    .输出OI为 StructObjectInspector 的实现类 StandardStructObjectInspector 的实例
         * .2.2.对于FINAL 和COMPLETE模式来说，是设定了方法terminate()返回值的OI实例
         *    .输出OI为 PrimitiveObjectInspector 的实现类 WritableDoubleObjectInspector 的实例
         */
        @Override
        public ObjectInspector init(Mode mode, ObjectInspector[] parameters)
                throws HiveException {
            assert (parameters.length == 1);
            super.init(mode, parameters);

            // init input
            if (mode == Mode.PARTIAL1 || mode == Mode.COMPLETE) {
                inputOI = (PrimitiveObjectInspector) parameters[0];
            } else {
                //部分数据作为输入参数时，用到的struct的OI实例，指定输入数据类型，用于解析数据
                soi = (StructObjectInspector) parameters[0];
                countField = soi.getStructFieldRef("count");
                sumField = soi.getStructFieldRef("sum");
                //数组中的每个数据，需要其各自的基本类型OI实例解析
                countFieldOI = (LongObjectInspector) countField.getFieldObjectInspector();
                sumFieldOI = (DoubleObjectInspector) sumField.getFieldObjectInspector();
            }

            // init output
            if (mode == Mode.PARTIAL1 || mode == Mode.PARTIAL2) {
                // The output of a partial aggregation is a struct containing
                // a "long" count and a "double" sum.
                //部分聚合结果是一个数组
                partialResult = new Object[2];
                partialResult[0] = new LongWritable(0);
                partialResult[1] = new DoubleWritable(0);
                /*
                 * .构造Struct的OI实例，用于设定聚合结果数组的类型
                 * .需要字段名List和字段类型List作为参数来构造
                 */
                ArrayList<String> fname = new ArrayList<String>();
                fname.add("count");
                fname.add("sum");
                ArrayList<ObjectInspector> foi = new ArrayList<ObjectInspector>();
                //注：此处的两个OI类型 描述的是 partialResult[] 的两个类型，故需一致
                foi.add(PrimitiveObjectInspectorFactory.writableLongObjectInspector);
                foi.add(PrimitiveObjectInspectorFactory.writableDoubleObjectInspector);
                return ObjectInspectorFactory.getStandardStructObjectInspector(fname, foi);
            } else {
                //FINAL 最终聚合结果为一个数值，并用基本类型OI设定其类型
                result = new DoubleWritable(0);
                return PrimitiveObjectInspectorFactory.writableDoubleObjectInspector;
            }
        }

        /**
         * 聚合数据缓存存储结构
         */
        static class AverageAgg implements AggregationBuffer {
            long count;
            double sum;
        }

        ;

        @Override
        public AggregationBuffer getNewAggregationBuffer() throws HiveException {
            AverageAgg result = new AverageAgg();
            reset(result);
            return result;
        }

        @Override
        public void reset(AggregationBuffer agg) throws HiveException {
            AverageAgg myagg = (AverageAgg) agg;
            myagg.count = 0;
            myagg.sum = 0;
        }

        boolean warned = false;

        /**
         * 遍历原始数据
         */
        @Override
        public void iterate(AggregationBuffer agg, Object[] parameters)
                throws HiveException {
            assert (parameters.length == 1);
            Object p = parameters[0];
            if (p != null) {
                AverageAgg myagg = (AverageAgg) agg;
                try {
                    //通过基本数据类型OI解析Object p的值
                    double v = PrimitiveObjectInspectorUtils.getDouble(p, inputOI);
                    myagg.count++;
                    myagg.sum += v;
                } catch (NumberFormatException e) {
                    if (!warned) {
                        warned = true;
                        LOG.warn(getClass().getSimpleName() + " "
                                + StringUtils.stringifyException(e));
                        LOG.warn(getClass().getSimpleName()
                                + " ignoring similar exceptions.");
                    }
                }
            }
        }

        /**
         * 得出部分聚合结果
         */
        @Override
        public Object terminatePartial(AggregationBuffer agg) throws HiveException {
            AverageAgg myagg = (AverageAgg) agg;
            ((LongWritable) partialResult[0]).set(myagg.count);
            ((DoubleWritable) partialResult[1]).set(myagg.sum);
            return partialResult;
        }

        /**
         * 合并部分聚合结果
         * 注：Object[] 是 Object 的子类，此处 partial 为 Object[]数组
         */
        @Override
        public void merge(AggregationBuffer agg, Object partial)
                throws HiveException {
            if (partial != null) {
                AverageAgg myagg = (AverageAgg) agg;
                //通过StandardStructObjectInspector实例，分解出 partial 数组元素值
                Object partialCount = soi.getStructFieldData(partial, countField);
                Object partialSum = soi.getStructFieldData(partial, sumField);
                //通过基本数据类型的OI实例解析Object的值
                myagg.count += countFieldOI.get(partialCount);
                myagg.sum += sumFieldOI.get(partialSum);
            }
        }

        /**
         * 得出最终聚合结果
         */
        @Override
        public Object terminate(AggregationBuffer agg) throws HiveException {
            AverageAgg myagg = (AverageAgg) agg;
            if (myagg.count == 0) {
                return null;
            } else {
                result.set(myagg.sum / myagg.count);
                return result;
            }
        }
    }

}
```

##### 2. 行转列(UDTF)

```java
package xin.carryzheng.hive.udtf.example;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.TaskExecutionException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ListObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.MapObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;

/**
 * GenericUDTFExplode.
 */
@Description(name = "explode",
        value = "_FUNC_(a) - separates the elements of array a into multiple rows,"
                + " or the elements of a map into multiple rows and columns ")
public class GenericUDTFExplode extends GenericUDTF {

    private ObjectInspector inputOI = null;

    @Override
    public void close() throws HiveException {
    }

    @Override
    public StructObjectInspector initialize(ObjectInspector[] args) throws UDFArgumentException {
        if (args.length != 1) {
            throw new UDFArgumentException("explode() takes only one argument");
        }

        ArrayList<String> fieldNames = new ArrayList<String>();
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<ObjectInspector>();

        switch (args[0].getCategory()) {
            case LIST:
                inputOI = args[0];
                fieldNames.add("col");
                fieldOIs.add(((ListObjectInspector)inputOI).getListElementObjectInspector());
                break;
            case MAP:
                inputOI = args[0];
                fieldNames.add("key");
                fieldNames.add("value");
                fieldOIs.add(((MapObjectInspector)inputOI).getMapKeyObjectInspector());
                fieldOIs.add(((MapObjectInspector)inputOI).getMapValueObjectInspector());
                break;
            default:
                throw new UDFArgumentException("explode() takes an array or a map as a parameter");
        }

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,
                fieldOIs);
    }

    private final Object[] forwardListObj = new Object[1];
    private final Object[] forwardMapObj = new Object[2];

    @Override
    public void process(Object[] o) throws HiveException {
        switch (inputOI.getCategory()) {
            case LIST:
                ListObjectInspector listOI = (ListObjectInspector)inputOI;
                List<?> list = listOI.getList(o[0]);
                if (list == null) {
                    return;
                }
                for (Object r : list) {
                    forwardListObj[0] = r;
                    forward(forwardListObj);
                }
                break;
            case MAP:
                MapObjectInspector mapOI = (MapObjectInspector)inputOI;
                Map<?,?> map = mapOI.getMap(o[0]);
                if (map == null) {
                    return;
                }
                for (Entry<?,?> r : map.entrySet()) {
                    forwardMapObj[0] = r.getKey();
                    forwardMapObj[1] = r.getValue();
                    forward(forwardMapObj);
                }
                break;
            default:
                throw new TaskExecutionException("explode() can only operate on an array or a map");
        }
    }

    @Override
    public String toString() {
        return "explode";
    }
}
```



##### 3. 使用分隔符连接字符串(UDF)

```java
package xin.carryzheng.hive.udf.example;

import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde.Constants;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.StringObjectInspector;
import org.apache.hadoop.io.Text;

/**
 * Generic UDF for string function
 * <code>CONCAT_WS(sep,str1,str2,str3,...)</code>. This mimics the function from
 * function_concat-ws
 *
 * @see org.apache.hadoop.hive.ql.udf.generic.GenericUDF
 */
@Description(name = "concat_ws", value = "_FUNC_(separator, str1, str2, ...) - "
        + "returns the concatenation of the strings separated by the separator.",
        extended = "Example:\n"
                + "  > SELECT _FUNC_('ce', 'fa', 'book') FROM src LIMIT 1;\n"
                + "  'facebook'")
public class GenericUDFConcatWS extends GenericUDF {
    private ObjectInspector[] argumentOIs;

    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
        if (arguments.length < 2) {
            throw new UDFArgumentLengthException(
                    "The function CONCAT_WS(separator,str1,str2,str3,...) needs at least two arguments.");
        }

        for (int i = 0; i < arguments.length; i++) {
            if (arguments[i].getTypeName() != Constants.STRING_TYPE_NAME
                    && arguments[i].getTypeName() != Constants.VOID_TYPE_NAME) {
                throw new UDFArgumentTypeException(i, "Argument " + (i + 1)
                        + " of function CONCAT_WS must be \"" + Constants.STRING_TYPE_NAME
                        + "\", but \"" + arguments[i].getTypeName() + "\" was found.");
            }
        }

        argumentOIs = arguments;
        return PrimitiveObjectInspectorFactory.writableStringObjectInspector;
    }

    private final Text resultText = new Text();

    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
        if (arguments[0].get() == null) {
            return null;
        }
        String separator = ((StringObjectInspector) argumentOIs[0])
                .getPrimitiveJavaObject(arguments[0].get());

        StringBuilder sb = new StringBuilder();
        boolean first = true;
        for (int i = 1; i < arguments.length; i++) {
            if (arguments[i].get() != null) {
                if (first) {
                    first = false;
                } else {
                    sb.append(separator);
                }
                sb.append(((StringObjectInspector) argumentOIs[i])
                        .getPrimitiveJavaObject(arguments[i].get()));
            }
        }

        resultText.set(sb.toString());
        return resultText;
    }

    @Override
    public String getDisplayString(String[] children) {
        assert (children.length >= 2);
        StringBuilder sb = new StringBuilder();
        sb.append("concat_ws(");
        for (int i = 0; i < children.length - 1; i++) {
            sb.append(children[i]).append(", ");
        }
        sb.append(children[children.length - 1]).append(")");
        return sb.toString();
    }
}
```



#### 三、在hive中使用自定义函数

##### 1. 将jar包上传到hive所在服务器

```shell
scp jarName.jar user@host:path # path为绝对路径
```

##### 2. 启动hive shell，将jar添加到classpath下

```shell
add jar path/jarName.jar # path为绝对路径
```

##### 3. 创建临时/永久函数

```shell
# functionName为hql函数名称，className为全类名
create temporary function functionName as 'className'; 
create function functionName as 'className';
```

##### 4.测试运行

![](http://pic.carryzheng.xin/zx_md/20190806153351.png)

