##  背景

项目开发中遇到一个表单，有很多相似的el-radio，随即打算用v-for实现，期间遇到了一些数据响应性丢失的问题，特此记录。

- 页面结构

```javascript
<div v-for="item in radioData">
                    <el-form-item :label="item.elFormItemLabel">
                        <el-radio-group v-model="item.vModel" @change="item.change">
                            <el-radio
                                v-for="itemR in item.elRadio"
                                :label="itemR.label"
                            >{{ itemR.value }}</el-radio>
                            <el-radio label="custom">
                                <span>自定义</span>
                                <el-input-number
                                    v-model="item.customVModel"
                                    :disabled="item.customDisabled"
                                    min="1"
                                    max="10000000"
                                    :controls="false"
                                />
                                <span>个</span>
                            </el-radio>
                        </el-radio-group>
                    </el-form-item>
                </div>
```

- 数据结构

```javascript
const radioData = reactive([
    {
        elFormItemLabel: '设备数',
        vModel: deviceCount,
        change: deviceCountChange,
        customVModel: deviceCountValue,
        customDisabled: deviceCountDisabled,
        elRadio: [
            {
                label: '100',
                value: 100
            },
            {
                label: '1000',
                value: 1000
            },
            //省略。。。
        ]
    }
])
```



### 问题1

一开始**radioData**没有用**reactive**包裹，导致radioData.vModel响应性丢失，猜想可能是radioData是常量，赋值进去以后就丢失响应性了，对象属性中用到响应性数据，对象也要定义成响应性。

### 问题2

```javascript
const form = reactive({
    deviceCountValue: 100,
    connectCountValue: 100,
});

const radioData = reactive(
    [
        {
            elFormItemLabel: '设备数',
            vModel: deviceCount,
            change: deviceCountChange,
            customVModel: form.deviceCountValue,
            customDisabled: deviceCountDisabled,
            //省略。。。
```

deviceCountValue是在 form里的一个属性，在radioData中为属性customVModel赋值form.deviceCountValue会导致customVModel丢失响应性，查看vue文档里对reactive有这么一段描述：“`reactive` 将解包所有深层的 [refs](https://v3.cn.vuejs.org/api/refs-api.html#ref)，同时维持 ref 的响应性。”于是改为下面的写法以维持响应性。

```javascript
const deviceCount = ref("100");
const deviceCountDisabled = ref(true);
const deviceCountValue = ref(100);
const form = reactive({
    deviceCountValue,
});

const radioData = reactive([
    {
        elFormItemLabel: '设备数',
        vModel: deviceCount,
        change: deviceCountChange,
        customVModel: deviceCountValue,
        customDisabled: deviceCountDisabled,
        //省略。。。
```



