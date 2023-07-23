from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(_name_)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///library.db'
db = SQLAlchemy(app)

class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    author = db.Column(db.String(100), nullable=False)
    quantity = db.Column(db.Integer, default=0)

class Member(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    outstanding_debt = db.Column(db.Float, default=0)

class Transaction(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    book_id = db.Column(db.Integer, db.ForeignKey('book.id'), nullable=False)
    member_id = db.Column(db.Integer, db.ForeignKey('member.id'), nullable=False)
    transaction_type = db.Column(db.String(10), nullable=False)  # 'issue' or 'return'
    transaction_date = db.Column(db.DateTime, nullable=False)
    rent_fee = db.Column(db.Float, default=0)

@app.route('/books', methods=['GET', 'POST'])
def handle_books():
    if request.method == 'GET':
        books = Book.query.all()
        result = [{"id": book.id, "name": book.name, "author": book.author, "quantity": book.quantity} for book in books]
        return jsonify(result)
    elif request.method == 'POST':
        data = request.get_json()
        new_book = Book(name=data['name'], author=data['author'], quantity=data['quantity'])
        db.session.add(new_book)
        db.session.commit()
        return jsonify({"message": "Book added successfully."})

@app.route('/books/<int:book_id>', methods=['GET', 'PUT', 'DELETE'])
def handle_book(book_id):
    book = Book.query.get_or_404(book_id)
    if request.method == 'GET':
        return jsonify({"id": book.id, "name": book.name, "author": book.author, "quantity": book.quantity})
    elif request.method == 'PUT':
        data = request.get_json()
        book.name = data['name']
        book.author = data['author']
        book.quantity = data['quantity']
        db.session.commit()
        return jsonify({"message": "Book updated successfully."})
    elif request.method == 'DELETE':
        db.session.delete(book)
        db.session.commit()
        return jsonify({"message": "Book deleted successfully."})

@app.route('/members', methods=['GET', 'POST'])
def handle_members():
    if request.method == 'GET':
        members = Member.query.all()
        result = [{"id": member.id, "name": member.name, "outstanding_debt": member.outstanding_debt} for member in members]
        return jsonify(result)
    elif request.method == 'POST':
        data = request.get_json()
        new_member = Member(name=data['name'])
        db.session.add(new_member)
        db.session.commit()
        return jsonify({"message": "Member added successfully."})

@app.route('/members/<int:member_id>', methods=['GET', 'PUT', 'DELETE'])
def handle_member(member_id):
    member = Member.query.get_or_404(member_id)
    if request.method == 'GET':
        return jsonify({"id": member.id, "name": member.name, "outstanding_debt": member.outstanding_debt})
    elif request.method == 'PUT':
        data = request.get_json()
        member.name = data['name']
        db.session.commit()
        return jsonify({"message": "Member updated successfully."})
    elif request.method == 'DELETE':
        db.session.delete(member)
        db.session.commit()
        return jsonify({"message": "Member deleted successfully."})

@app.route('/transactions', methods=['POST'])
def handle_transaction():
    data = request.get_json()
    book_id = data['book_id']
    member_id = data['member_id']
    transaction_type = data['transaction_type']

if _name_ == '_main_':
    db.create_all()
    app.run(debug=True)
