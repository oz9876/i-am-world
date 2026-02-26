#日常开发


  可以直接按 README 里的步骤来跑，我帮你用中文简单总结一下本地运行方式（不走 Docker）：
  一、安装依赖
  在项目根目录执行：

  ## 建议先创建虚拟环境，这步可选
  python -m venv venv
  source venv/bin/activate  # Windows 用 venv\Scripts\activate

  ## 后端依赖
  pip install -r requirements.txt

  ## 前端依赖（需要 Node.js）
  cd web
  npm install
  cd ..

  二、配置 LLM
  最简单是先用前端里的界面配置（运行后在网页右上角/设置里填 API 信息）。
  如果你想提前写配置文件，在项目根目录创建或编辑 static/local_config.yml：

  llm:
    base_url: https://api.deepseek.com
    key: your-api-key-here
    model_name: deepseek-chat
    fast_model_name: deepseek-chat
    mode: default

  把 your-api-key-here 换成你自己的 key，或者按你实际用的服务改。
  三、启动服务（推荐开发模式）
  在项目根目录执行：

  source .venv/bin/activate
  python src/server/main.py --dev

  正常的话浏览器会自动打开前端页面；如果没自动打开，可以手动访问：
  前端地址: http://localhost:8123
  后端 API: http://localhost:8002

  如果你更想用 Docker 一键跑，在项目根目录执行：
  docker-compose up -d --build
  如果你现在照做遇到报错，把报错信息贴出来，我帮你一起排。




# 关于同步原仓库更新

  🔄 同步原仓库更新

  当原仓库更新时：

  1️⃣ 拉取上游代码

  git fetch upstream

  2️⃣ 合并到你的主分支

  git checkout main
  git merge upstream/main

  3️⃣ 再把 main 合并到你的开发分支

  git checkout my-dev
  git merge main

  这样你的开发分支就带上了最新的原仓库更新。
