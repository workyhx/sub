> 1. 创建一个`.py`文件，并把下列代码写入该文件
> 2. 使用`pip`安装`flask`
> 3. 同级目录下创建`sub.txt`文件，并且将链接每行写入一个
> 4. `/root/ygkkkca/cert.crt`和`/root/ygkkkca/private.key`为证书，需要创建证书
> 5. 最后访问`https://你的IP:7000`即可获取订阅
```py
# 导入Flask核心模块和base64编码模块
import ssl
from flask import Flask
import base64

# 初始化Flask应用实例
app = Flask(__name__)

# 定义根路由（访问https://IP:7000 即可触发该函数）
@app.route('/')
def return_base64_subtxt():
    """每次访问该路由时，读取sub.txt并返回其Base64编码内容"""
    try:
        # with上下文管理器：自动释放文件资源，每次访问重新读取
        with open("sub.txt", "rb") as f:
            file_content = f.read()
        
        # Base64编码并转换为UTF-8字符串返回
        base64_encoded = base64.b64encode(file_content)
        base64_str = base64_encoded.decode("utf-8")
        
        return base64_str
    except FileNotFoundError:
        return "错误：当前脚本同级目录下未找到 sub.txt 文件，请先创建该文件！", 404
    except PermissionError:
        return "错误：没有读取 sub.txt 文件的权限！", 500
    except Exception as e:
        return f"错误：读取文件失败，异常信息：{str(e)}", 500

# 程序入口
if __name__ == '__main__':
    # 配置SSL证书路径（你指定的公钥和密钥路径）
    # ssl_cert_path = "/root/ygkkkca/cert.crt"  # 证书公钥文件
    # ssl_key_path = "/root/ygkkkca/private.key"  # 证书密钥文件
    # 自定义SSL上下文，关闭证书验证（测试用）
    context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
    context.load_cert_chain(
        certfile="/root/ygkkkca/cert.crt",
        keyfile="/root/ygkkkca/private.key"
    )
    # 关闭证书验证（仅测试环境用！）
    context.check_hostname = False
    context.verify_mode = ssl.CERT_NONE



    # 启动Flask HTTPS服务
    # ssl_context：指定证书文件，启用HTTPS功能，仅允许HTTPS访问
    # host='0.0.0.0'：支持局域网访问，可根据需要改为127.0.0.1（仅本地访问）
    # port=7000：监听7000端口（HTTPS服务端口）
    # debug=False：生产环境推荐关闭，避免安全风险
    app.run(
        host='0.0.0.0',
        port=7000,
        debug=False,
        ssl_context=context  # 核心：配置SSL证书，启用HTTPS
    )
```
