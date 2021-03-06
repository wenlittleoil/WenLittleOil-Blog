## 1
iphone微信使用base64显示图片，不支持background-image，应使用img的src显示。

## 2
点击底部按钮时,滚动至顶  
this.pageTopEle为页顶元素
```
if (this.pageTopEle) {
  // 不兼容时降级
  if (this.pageTopEle.scrollIntoView) {
    /** 平滑滚动置顶 */
    this.pageTopEle.scrollIntoView({ block: 'start', behavior: 'smooth' });
  } else {
    /** 生硬滚动置顶 */
    const pageTop = this.pageTopEle.offsetTop;
    window.scrollTo(0, pageTop);
  }
}
```

## 3
对于移动端的touch系列事件(touchstart/touchend/touchcancel等)，不支持冒泡，其事件处理程序必须直接注册在dom树最底层的img元素上。并且对于长按事件的日志统计，为了解决android和ios下的兼容，touchend和touchcancel必须同时注册，因为手指离开屏幕后只会触发其中一个。demo如下：  
```
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import collectLog from '@/collectLog';

class ImgWithPressLog extends Component {
  constructor(props) {
    super(props);
    this.state = {};
    this.longTouch = false;
  }
  imgTouchStart = event => {
    this.longTouch = false;
    setTimeout(() => (this.longTouch = true), 500);
    const { onTouchStart } = this.props;
    this.extraEventHandler(onTouchStart, event);
  };
  imgTouchEnd = event => {
    this.handleTouchCollect();
    const { onTouchEnd } = this.props;
    this.extraEventHandler(onTouchEnd, event);
  };
  imgTouchCancel = event => {
    this.handleTouchCollect();
    const { onTouchCancel } = this.props;
    this.extraEventHandler(onTouchCancel, event);
  };
  extraEventHandler = (...args) => {
    const func = args[0];
    const params = args.slice(1);
    if (typeof func === 'function') {
      func.apply(this, params);
    }
  };
  handleTouchCollect = () => {
    if (this.longTouch) {
      const { logParams, } = this.props;
      if (logParams && typeof logParams === 'object') {
        collectLog(logParams);
        this.longTouch = false;
        return;
      }
    }
    this.longTouch = false;
  };
  render() {
    const {
      onTouchStart,
      onTouchEnd,
      onTouchCancel,
      alt,
      ...otherProps
    } = this.props;
    return (
      <img
        onTouchStart={event => this.imgTouchStart(event)}
        onTouchEnd={event => this.imgTouchEnd(event)}
        onTouchCancel={event => this.imgTouchCancel(event)}
        alt={alt}
        {...otherProps}
      />
    );
  }
}

ImgWithPressLog.propTypes = {
  logParams: PropTypes.object,
};

export default ImgWithPressLog;

```

## 4  
iphone7某些ios微信版本中，微信网页api授权不支持h5路由的history模式，若使用history模式，页面路由发生跳转后会导致授权失败，从而导致无法调起微信上传图片等微信api。解决方式为：在当前的路由页面内重新调用后端接口wxconf，获取授权数据后在前端再调用wx.config获取jssdk的api授权验证。

## 5
移动端页面悬浮底部的固定定位按钮。
使用如下错误示例的css书写方式会导致在ios微信中，首次点击按钮失效，点击第二次才有效果的bug.

> 错误示例：
```
position: fixed;
left: 0;
right: 0;
bottom: 0;
```
> 正确示例：
```
position: fixed;
left: 50%;
transform: translateX(-50%);
bottom: 0;
```
## 6
ctrl + c 或 菜单键复制 的原生copy事件对pc端支持不错，对m端支持性不佳。m端可使用三方库clipboard或react-copy-to-clipboard来解决。

## 7
mysql数据表中，以bigint(20) 数据类型存储订单号（例如201902191734473812），由于js精度限制，获取出来的数据会出现失准的异常情况.
解决方式为: 查询数据表时使用cast函数将数据类型转变为char(64)的字符串形式。
示例：
```
  SELECT
  id, `CAST(exchange_id AS CHAR(64)) AS exchange_id`, exchange_title, amount, create_time,
  contact_name, contact_phone, address, status
  FROM exchange
  WHERE user_id = :studentId
  AND status != -1
  ORDER BY create_time DESC
  LIMIT :offset, :pageSize;
```

