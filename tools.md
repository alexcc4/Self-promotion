## Python

- [Flask-jwt-extended](http://flask-jwt-extended.readthedocs.io/en/latest/) 在 Flask 上使用 JWT 的库  
- [Flask-restful](https://flask-restful.readthedocs.io/en/0.3.5/) Flask 实现 Restful 的库  
- [Flask-Migrate](http://flask-migrate.readthedocs.io/en/latest/) Flask 管理数据库迁移的库，
 基于 [alembic](http://alembic.zzzcomputing.com/en/latest/) alembic 的 migrate 功能强大，但是存在: 
     * 对于索引，主键外键等迁移支持度不够
     * 对于 postgresql 的 Enum, JSON 类型支持不太友好
     * migrate 若需要删除有依赖的表，很多时候是无法成功删除      
 
- [Flask-redis](https://github.com/underyx/flask-redis) Flask 集成 redis 的库。 多用来与 jwt 库结合，实现登录登出。  
  * [redis-py](https://github.com/andymccurdy/redis-py) 这个库现在还没有正式支持 namespace 这个特性（namespace 可以区别 key， 数据的安全）https://github.com/andymccurdy/redis-py/issues/12
  
- [Flask-testing](https://pythonhosted.org/Flask-Testing/) Flask 集成 test 的框架
- [Flask-Script](https://flask-script.readthedocs.io/en/latest/) Flask 命令行库 
- [Faker](https://github.com/joke2k/faker) fake data 库
- [marshmallow](https://github.com/marshmallow-code/marshmallow) 序列化、反序列化以及表单检验 （用来替换 flask-restful 自带的 parse input/ouput 库）
## 运维

- 服务器配置 https（在 http 网站在国内备案比较长、麻烦的情况下，快速配置 https,会带来很多便利 ^_^）  
  * [acme](https://github.com/Neilpang/acme.sh)
- [vagrant](https://www.vagrantup.com/)
- [ansible](https://www.ansible.com/)  
- [docker](https://www.docker.com/)
