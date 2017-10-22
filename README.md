# ueditor_with_rails

# User Guiding

## 1. Ueditor with rails. I created this project because Ueditor with rails can not be found on the internet and on the official site.

### 1.1 Step 1. Put directory of the ueditor under ``/public/plugins/`` under your project. The default controller name is ``resource_controller.rb``. You can custom it with ``ueditor.config.js`` if it doesn't suite your project.

### 1.2 Step 2. You can custom more settings with ``config.json`` of the ueditor. I will show my ``resource_controller.rb`` below that you can find in this repo.

```ruby
#encoding:utf-8
require 'json'
require 'tempfile'
require 'base64'
#用于上传项目相关的资源
class ResourceController < ApplicationController
  #ueditor的配置
  def handle_file
    #ueditor是通过在url中的传入ueditor_action（原本为action，但是由于其与rails冲突，所以我将其改为了ueditor_action）字段来区分具体的操作的
    return if params[:ueditor_action].blank?
    cur_action = params[:ueditor_action]

    #刚进入页面时editor会进行config的访问
    if (cur_action == "config")
      #打开config.json文件，将其返回，注意，我这里是将config.json文件放在/public/plugins/ueditor/目录下，可以自己的需求，对这里进行相应的更改
      json = File.read("#{Rails.root.to_s}/public/plugins/ueditor/config.json")
      #正则替换，使其成为ueditor可以识别的格式
      json = json.gsub(/\/\*[\s\S]+?\*\//, "")
      result = JSON.parse(json).to_s
      result = result.gsub(/=>/,":")
      #返回结果
      render :text => result
    #图片上传
    elsif (cur_action == "upload_image")
      upload_image_video
    #视频上传
    elsif (cur_action == "upload_video")
      upload_image_video
    #涂鸦上传
    elsif (cur_action == "upload_scrawl")
      upload_scrawl
    else
      respond_result
    end
  end


private
  #涂鸦文件的处理，ueditor使用base64编码，并且为png格式
  def upload_scrawl
    status = 'SUCCESS'
    if params[:upfile].blank?
      return
    end
    scrawl_base64 = params[:upfile]
    tempfile = Tempfile.new("upload_scrawl.png")
    tempfile.binmode
    tempfile.write(Base64.decode64(scrawl_base64))
    tempfile.close
    #开始保存文件
    filename = get_random_string(10) + "_" + get_random_string(10) + "_" + get_random_string(10) + ".png"
    
    #保存文件到项目指定的路径，该方法未实现，需要自己去实现
    save_file(tempfile,filename)
    
    respond_result(filename,status)
  end


  #上传图片和视频的处理
  def upload_image_video
    status = 'SUCCESS'
    #对视频文件或者图片文件进行保存，需要自己实现
    respond_result(filename,status)
  end


  #负责向客户端返回数据
  def respond_result(filename='', status='')
    #该变量是根据ueditor自带的demo写的，不知道为什么，demo没有也没有传这个字段
    callback = params[:callback]
    response_text = ''
    #构造需要返回的数据，这个是ueditor已经约定好的，不能随意对字段进行修改。也不能使用rails内置的render :json语句，因为这样最后得到的数据格式是无法被ueditor解析的。
    if filename.blank?
      response_text = "{\"name\":\"\"," +
        "\"originalName\":\"\"," +
        "\"size\":\"\",\"state\":\"#{status}\",\"type\":\"\",\"url\":\"\"}"
    else
      response_text = "{\"name\":\"#{filename}\"," +
          "\"originalName\":\"#{filename}\"," +
          "\"size\":\"#{File.size(TalentUtils.get_upload_file(filename)).to_s}\",\"state\":\"#{status}\",\"type\":\"#{File.extname(filename)}\",\"url\":\"#{filename}\"}"
    end

    if callback.blank?
      render :text => response_text
    else
      render :text => "<script>"+ callback +"(" + response_text + ")</script>"
    end
  end


  #生成随机的字符串
  def get_random_string(num = 5)
    #5是指生成字符串的个数，默认为5
    rand(36 ** num).to_s(36)
  end
end
```



## 2 More infomations you can get in my <a href="http://www.cnblogs.com/seancheer/p/5488971.html">blog</a>.