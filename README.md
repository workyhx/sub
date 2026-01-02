# 下列代码复制为`.py`文件，然后安装flask，直接运行即可，还需要在`.py`文件同级目录下创建`sub.txt`文件，里面每行填写一个链接，即可实现订阅
```py
# 导入Flask核心模块和base64编码模块
from flask import Flask
import base64

# 初始化Flask应用实例，__name__是Flask的默认应用标识
app = Flask(__name__)


# 定义根路由（访问http://IP:7000 即可触发该函数）
@app.route('/')
def return_base64_subtxt():
    """每次访问该路由时，读取sub.txt并返回其Base64编码内容"""
    # 核心：使用with上下文管理器读取文件
    # 1. rb模式：二进制读取，适配所有类型文件（避免文本编码异常）
    # 2. with语句：读取完成后自动关闭文件句柄，释放文件资源，无需手动调用f.close()
    # 3. 每次访问都会执行该读取逻辑，确保是"新的读取"
    try:
        with open("sub.txt", "rb") as f:
            file_content = f.read()  # 一次性读取文件所有二进制内容

        # 对文件二进制内容进行Base64编码，返回二进制格式编码结果
        base64_encoded = base64.b64encode(file_content)
        # 将二进制编码结果转换为UTF-8字符串，便于HTTP响应返回
        base64_str = base64_encoded.decode("utf-8")

        # 返回Base64编码字符串给客户端
        return base64_str
    except FileNotFoundError:
        # 处理sub.txt文件不存在的异常，返回友好提示
        return "错误：当前脚本同级目录下未找到 sub.txt 文件，请先创建该文件！", 404
    except PermissionError:
        # 处理文件读取权限不足的异常
        return "错误：没有读取 sub.txt 文件的权限！", 500
    except Exception as e:
        # 处理其他未知读取异常
        return f"错误：读取文件失败，异常信息：{str(e)}", 500


# 程序入口
if __name__ == '__main__':
    # 启动Flask服务，监听7000端口
    # host='0.0.0.0'：允许局域网内其他设备访问（仅本地访问可省略，默认127.0.0.1）
    # debug=False：生产环境推荐关闭，避免安全风险（开发时可改为True，热重载）
    app.run(host='0.0.0.0', port=7000, debug=False)
```
