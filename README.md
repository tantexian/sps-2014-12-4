sps-2014-12-4
=============

类似nova的一个openstack服务demo

======
sps
======

sps is a project that defines services for discovering, registering,
retrieving and storing virtual machine demos. Use the following resources
to learn more:

* `Official sps documentation <http://docs.openstack.org/developer/sps/>`_
* `Official Client documentation <http://docs.openstack.org/developer/python-spsclient/>`_

1 ./install.sh  #此脚本用来安装sps服务，并启动、内容如下
==========================================================================
rm -rf build/
rm -rf /usr/lib/python2.6/site-packages/sps/
rm -rf /usr/bin/sps-*
python setup.py install
rm -rf /etc/sps/*
cp -r ./etc/* /etc/sps/
==========================================================================

==========================================================================

2、#数据库表增加(参照如下Demo数据库表的格式定义)
./sps/db/sqlalchemy/models.py
==========================================================================
class Demo(BASE, SpsBase):
    """Represents an task in the datastore"""
    __tablename__ = 'demos'
    __table_args__ = (Index('ix_tasks_type', 'type'),
                      Index('ix_tasks_status', 'status'),
                      Index('ix_tasks_deleted', 'deleted'),
                      Index('ix_tasks_updated_at', 'updated_at'))

    id = id = Column(Integer, primary_key=True)
    name = Column(String(30))
    type = Column(String(30))
    status = Column(String(30))
==========================================================================

3 .sps-manage db_sync #生成数据库（同步数据库表到数据库，）


4 . #启动服务
==========================================================================
ps aux | grep sps | grep -v grep | awk '{print $2}' | xargs kill -9
sps-api --debug
==========================================================================



5. #数据库的增删查改API
==========================================================================
增：
curl -i -H 'Content-Type: application/json' -H 'User-Agent: python-spsclient' -X 'POST' http://192.168.10.31:9898/v1/demos -d '{"demo": {"name": "ttx", "type": "1"}}'

查all：
curl -i -H 'Content-Type: application/json' -H 'User-Agent: python-spsclient' -X 'GET' http://192.168.10.31:9898/v1/demos

查某个：
curl -i -H 'Content-Type: application/json' -H 'User-Agent: python-spsclient' -X 'GET' http://192.168.10.31:9898/v1/demos/1

更新：
curl -i -H 'Content-Type: application/json' -H 'User-Agent: python-spsclient' -X 'PUT' http://192.168.10.31:9898/v1/demos/25 -d '{"demo": {"name": "ttx---25--", "type": "1"}}'

删
curl -i -H 'Content-Type: application/json' -H 'User-Agent: python-spsclient' -X 'DELETE' http://192.168.10.31:9898/v1/demos/1



6、 #发布route API：
./sps/api/v1/router.py
==========================================================================
class API(wsgi.Router):

    """WSGI router for sps v1 API requests."""

    def __init__(self, mapper):
        demos_resource = demos.create_resource()

        mapper.connect("/",
                       controller=demos_resource,
                       action="index")
        mapper.connect("/demos",
                       controller=demos_resource,
                       action='index',
                       conditions={'method': ['GET']})
        mapper.connect("/demos",
                       controller=demos_resource,
                       action='create',
                       conditions={'method': ['POST']})
        mapper.connect("/demos/detail",
                       controller=demos_resource,
                       action='detail',
                       conditions={'method': ['GET', 'HEAD']})
        mapper.connect("/demos/{id}",
                       controller=demos_resource,
                       action="meta",
                       conditions=dict(method=["HEAD"]))
        mapper.connect("/demos/{id}",
                       controller=demos_resource,
                       action="show",
                       conditions=dict(method=["GET"]))
        mapper.connect("/demos/{id}",
                       controller=demos_resource,
                       action="update",
                       conditions=dict(method=["PUT"]))
        mapper.connect("/demos/{id}",
                       controller=demos_resource,
                       action="delete",
                       conditions=dict(method=["DELETE"]))

        #add you API route at this ,ex: define you demos2_resource = demos2.create_resource()

        super(API, self).__init__(mapper)
==========================================================================


7、sql CURD（增删查改）
./sps/db/sqlalchemy/api.py
==========================================================================
def get_demo(context, id):
    session = get_session()
    query = model_query(context, models.Demo, session=session, read_deleted="no")
    result = query.filter_by(id=id).first()
    return  result

def get_demo_list(context):
    session = get_session()
    query = model_query(context, models.Demo, session=session, read_deleted="no")
    result = query.all()

    return result

def add_demo(context, demo):
    try:
        demo_ref = models.Demo()
        for key, val in demo.items():
            demo_ref.update({key: val})
        demo_ref.update({'created_at': timeutils.datetime.datetime.utcnow()})
        demo_ref.save()
    except:
        raise exception.DemoCanNotDelete(id=id)
    return demo_ref

def delete_demo(context, id):
    try:
        session = get_session()
        query = model_query(context, models.Demo, session=session, read_deleted="no")
        query.filter_by(id=id).first().soft_delete(session)
    except:
        raise exception.DemoCanNotDelete(id=id)

def update_demo(context, id, demo):
    try:
        session = get_session(autocommit=True)
        query = model_query(context, models.Demo, session=session, read_deleted="no")
        demo_ref = query.filter_by(id=id).first()
        if not demo_ref:
            raise exception.DemoNotFound(id=id)
        for key, val in demo.items():
            demo_ref.update({key: val})
        demo_ref.update({'updated_at': timeutils.datetime.datetime.utcnow()})
        session.add(demo_ref)
        session.flush()
    except:
        raise exception.DemoCanNotUpdatate(id=id)
    return  demo_ref
==========================================================================