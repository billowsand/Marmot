===========================
在Glance中加入parent_id选项
===========================

修改数据库
==========

在下面目录中创建文件"035_add_parent_id"::

 glance.db.sqlalchemy.migrate_repo.versions

添加如下代码::

 import sqlalchemy


 def upgrade(migrate_engine):
     meta = sqlalchemy.MetaData()
     meta.bind = migrate_engine

     images = sqlalchemy.Table('images', meta, autoload=True)
     parent_id = sqlalchemy.Column('parent_id',
                                     sqlalchemy.String(36))
     images.create_column(parent_id)


 def downgrade(migrate_engine):
     meta = sqlalchemy.MetaData()
     meta.bind = migrate_engine

     images = sqlalchemy.Table('images', meta, autoload=True)
     images.columns['parent_id'].drop()

修改访问接口
===========

