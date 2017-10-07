#!/usr/bin/env python3
# -*- coding:utf-8 -*-

from flask import Flask, render_template
import os, json
from flask.ext.sqlalchemy import SQLAlchemy
from datetime import datetime
from pymongo import MongoClient

app = Flask(__name__)
app.config.update(dict(SQLALCHEMY_DATABASE_URI = 'mysql://root@localhost/shiyanlou'))
app.config['TEMPLATES_AUTO_RELOAD'] = True
db = SQLAlchemy(app)

client = MongoClient('127.0.0.1', 27017)
mongo_db = client.shiyanlou

'''
jsons_name = os.listdir("/home/shiyanlou/files")
with open(os.path.join("/home/shiyanlou/files",jsons_name[0]), 'r') as f:
    world = json.loads(f.read())
with open(os.path.join("/home/shiyanlou/files",jsons_name[1]), 'r') as f:
    shiyanlou = json.loads(f.read())
'''

class File(db.Model):
    __tablename__ = 'file'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(80))
    content = db.Column(db.Text)
    cteated_time = db.Column(db.DateTime)

    category_id = db.Column(db.Integer, db.ForeignKey('category.id'))
    category = db.relationship('Category',
        backref=db.backref('files', lazy='dynamic'))

    def __init__(self, title, created_time, category, content):
        self.title = title
        if created_time is None:
            created_time = datetime.utcnow()
        self.created_time = created_time
        self.category = category
        self.content = content

    def add_tag(self, tag_name):
        file_item = mongo_db.files.find_one({'file_id': self.id})
        if file_item:
            tags = file_item['tags']
            if tag_name not in tags:
                tags.append(tag_name)
            mongo_db.files.update_one({'file_id': self.id}, {'$set': {'tags': tags}})
        else:
            tags = [tag_name]
            mongo_db.files.insert_one({'file_id': self.id, 'tags': tags})
        return tags

    def remove_tag(self, tag_name):
        file_item = mongo_db.files.find_one({'file_id': self.id})
        if file_item:
            tags = file_item['tags']
            try:
                new_tags = tags.remove(tag_name)
            except ValueError:
                return tags
            mongo_db.files.update_one({'file_id': self.id}, {'$set', {'tags': new_tags}})
            return new_tags
        return []

    @property
    def tags(self):
        file_item = mongo_db.files.find_one({'file_id':self.id})
        if file_item:
            return file_item['tags']
        else:
            return []

    def __repr__(self):
        return '<File %r>' % self.title


class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80))

    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return '<Category %r>' % self.name


def insert_datas():
    java = Category('Java')
    python = Category('Python')
    file1 = File('Hello Java', datetime.utcnow(), java, 'File Content - Java is cool!')
    file2 = File('Hello Python', datetime.utcnow(), python, 'File Content - Python is cool!')
    db.session.add(java)
    db.session.add(python)
    db.session.add(file1)
    db.session.add(file2)
    db.session.commit()
    file1.add_tag('tech')
    file1.add_tag('java')
    file1.add_tag('linux')
    file2.add_tag('tech')
    file2.add_tag('python')



   


db.create_all()
 
f = db.session.query(File).all()
if not f:
    insert_datas()
c = db.session.query(Category).all()

@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'),404

@app.route('/')
def index():
    return render_template('index.html',file = f )

@app.route('/files/<file_id>')
def file(file_id):  
     return render_template('file.html',file_id = file_id, file = f, category = c, )


if __name__=='__main__':
       
    app.run()
   
