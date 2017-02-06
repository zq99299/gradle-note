# 日志
简单说就是编写脚本的时候可以使用日志，类似java中的slf4j使用。
```groovy
// 安静的
logger.quiet('An info log message which is always logged.')
logger.error('An error log message.')
logger.warn('A warning log message.')
// 生命周期
logger.lifecycle('A lifecycle info log message.')
logger.info('An info log message.')
logger.debug('A debug log message.')
logger.trace('A trace log message.')
```