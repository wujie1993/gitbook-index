# 开源项目管理

## 提交流程

1. 如果是他人的项目，先对项目进行fork
2. 如果要提交的内容属于可描述的问题，使用如下模板创建issue

   ```text
   <!-- **Are you in the right place?**
   1. For issues or feature requests, please create an issue in this repository.
   2. Did you already search the existing open issues for anything similar? -->


   **Bug Report**

   What happened:

   What you expected to happen:

   How to reproduce it (minimal and precise):
   <!-- Please let us know any circumstances for reproduction of your bug. -->

   Share your codes of demo

   **Environment**:
   * OS (e.g. from /etc/os-release):
   * Kernel (e.g. `uname -a`):

   ```

3. 如果要提交的内容属于可描述的特性，使用如下模板创建issue

   ```text
   **Is your feature request related to a problem? Please describe.**
   A clear and concise description of what the problem is. Ex. I'm always frustrated when [...]

   **Describe the solution you'd like**
   A clear and concise description of what you want to happen.

   **Describe alternatives you've considered**
   A clear and concise description of any alternative solutions or features you've considered.

   **Additional context**
   Add any other context or screenshots about the feature request here.

   ```

4. 如果要提交的内容是微小的调整，则无需创建issue
5. 根据需要提交的内容在fork下来的项目master分支中创建新分支，如：fix-readme-typo，feature-support-user-login
6. 将内容commit到新建的分支中，在push前对commit进行必要的合并，保证提交树的简洁
7. 在确认已将所有内容变更提交完成后，向源项目提交pull requrest
8. 在pull request中会有ci流程对提交的内容进行检查，检查通过后会分配给源项目的reviewer再次检查，在这期间对于新分支进行提交会再次触发ci流程
9. reviewer检查通过后会对pull request执行merge操作，将新分支中的提交合并到master分支中，在合并完成后对于新分支的提交不会再被识别

## 提交规范

1. 一个commit对应处理一个问题，如果对于一个小问题做了多次commit，在push前需要对commit进行合并
2. 每个commit使用如下格式

   ```text
   {{ 模块简称 }} {{ issue编号 }}: {{ 提交内容简介 }}

   {{ 提交内容描述 }}

   Signed-off-by: {{ 提交者名称 }} {{ 提交者邮箱 }}
   ```

3. 如果处理的问题较大，涉及多个模块的变更，视模块间是否存在关联。如果存在关联，则将多个模块的commit合并为一个commit，如果不存在关联，则以模块为单位分开为多个commit。



