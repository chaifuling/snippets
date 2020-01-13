# 实现阿里云OSS图片处理方法

阿里云OSS提供了便捷的方式处理图片的旋转、缩放、格式转换...等等操作，如果我们每次通过字符串拼接的方式来处理的话，需要判断的条件很多，而且容易出错。写出你的方法，帮助自己以后在项目中快速得到需要的图片地址。

接口方式不限（重点）
支持oss对应的所有操作
考虑原始图片地址带有查询参数
考虑原始图片地址带有oss处理参数
不考虑浏览器兼容性

阿里OSS图片处理文档： https://help.aliyun.com/document_detail/44686.html

```js

class NoKeyError extends Error {}

const throwNoKeyError = method => {
  throw new NoKeyError(`方法：${method}，需要至少一个参数`);
};

function generateMethods() {
  'resize,blur,circle,crop,indexcrop,rotate,bright,contrast,sharpen,format,watermark,interlace,quality,roundedCorners,autoOrient'
    .split(',')
    .forEach(method => {
      OssUrl.prototype[method] = function(k = throwNoKeyError(), v) {
        if (typeof k === 'object') {
          this[formatObjKey][method] = k;
        } else {
          if (!this[formatObjKey][method]) this[formatObjKey][method] = {};
          this[formatObjKey][method][k] = v;
        }
        return this;
      };
    });
}

// 用a拼接obj里面的每对kv，并用b分割
function joinStr(obj, a, b) {
  return Object.keys(obj)
    .reduce((acc, cur) => {
      if (obj[cur]) return (acc += `${b}${cur}${a}${obj[cur]}`);
      return (acc += `${b}${cur}`);
    }, '')
    .slice(1);
}

// 驼峰转中横线
const decamelize = str => str.replace(/\B([A-Z])/g, '-$1').toLowerCase();

const formatObjKey = Symbol('formatObjKey');

class OssUrl {
  [formatObjKey] = {};

  constructor(origin) {
    this.origin = origin;
  }

  get url() {
    return this.buildURL();
  }

  buildURL() {
    const url = new URL(this.origin);
    const formatParams = Object.keys(this[formatObjKey]).reduce(
      (acc, method) => {
        const str = joinStr(this[formatObjKey][method], '_', ',');
        if (!str) return acc;
        return (acc += `/${decamelize(method)},${str}`);
      },
      ''
    );
    url.searchParams.set('x-oss-process', `image${formatParams}`);
    return url.href;
  }
}

generateMethods();

// 对外暴露方法
function ossImg(origin) {
  return new OssUrl(origin);
}

// test

ossImg('http://image-demo.oss-cn-hangzhou.aliyuncs.com/example.jpg').resize({w:200,h:500})
            .blur('r',3)
            .blur('s',2)
            .format('webp')


```