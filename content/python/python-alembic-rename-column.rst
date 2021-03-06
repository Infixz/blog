[python]通过 alembic 重命名表列名（Model字段名）
============================================================

:date: 2014-11-25
:tags: alembic, sqlalchemy
:slug: python-alembic-rename-column-name-field-name


为什么要特意讲一下这个，是因为 alembic 无法自动检测是否对字段名进行了重命名操作，需要我们手动添加相关语句。

假设有下面这样一个 Model

.. code-block:: python

    class User(Base):
        __tablename__ = 'user'

        id = Column(Integer, primary_key=True)
        name = Column(String(20))

我们现在想要把 ``name`` 字段改名为 ``username``, 通过 ``alembic revision --autogenerate`` 无法实现我们的目的::

    INFO  [alembic.migration] Context impl MySQLImpl.
    INFO  [alembic.migration] Will assume non-transactional DDL.
    INFO  [alembic.autogenerate.compare] Detected added column 'user.username'
    INFO  [alembic.autogenerate.compare] Detected removed column 'user.name'
      Generating /home/xx/alembic/versions/4475103466d3_rename_user_name_to_username.py .
    .. done

上面的检测结果明显不是我们想要的，需要手动更改生成的 version 文件:

.. code-block:: python

    from sqlalchemy.dialects import mysql

    def upgrade():
        ### commands auto generated by Alembic - please adjust! ###
        # 把 user 表里的 name 列重命名为 username
        op.alter_column('user', 'name', new_column_name='username',
                        existing_type=mysql.VARCHAR(length=20))
        ### end Alembic commands ###


    def downgrade():
        ### commands auto generated by Alembic - please adjust! ###
        op.alter_column('user', 'username', new_column_name='name',
                        existing_type=mysql.VARCHAR(length=20))
        ### end Alembic commands ###

``alemblic upgrade`` 命令结果::

    $ alembic upgrade 549c504fe1b5:4475103466d3 --sql
    INFO  [alembic.migration] Context impl MySQLImpl.
    INFO  [alembic.migration] Generating static SQL
    INFO  [alembic.migration] Will assume non-transactional DDL.
    INFO  [alembic.migration] Running upgrade 549c504fe1b5 -> 4475103466d3, rename User.name to username
    -- Running upgrade 549c504fe1b5 -> 4475103466d3

    ALTER TABLE user CHANGE name username VARCHAR(20) NULL;

    UPDATE alembic_version SET version_num='4475103466d3';

可以看到生成的 sql 语句就是我们想要的结果。

通过 ``alter_column`` 方法进行重命名的基本用法就是 ``alter_column('表名', '先有的列名', new_column_name='重命名后的列名', existring_type=字段类型)``

通过 ``alter_column`` 方法可以更多的 ``ALTER TABLE`` 操作，详见 `文档`__

__ http://alembic.readthedocs.org/en/latest/ops.html#alembic.operations.Operations.alter_column


参考
------

* `Operation Reference — Alembic 0.7.0 documentation`__

__ http://alembic.readthedocs.org/en/latest/ops.html#alembic.operations.Operations.alter_column
