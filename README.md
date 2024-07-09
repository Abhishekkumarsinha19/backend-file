# backend-file
library_management/
│
├── app.py
├── init_db.py
└── requirements.txt
#Flask
import sqlite3

# Connect to the SQLite database
conn = sqlite3.connect('library.db')
cursor = conn.cursor()

# Create the books table
cursor.execute('''
CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    published_date TEXT NOT NULL,
    isbn TEXT NOT NULL
)
''')

# Commit the changes and close the connection
conn.commit()
conn.close()
from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

# Function to connect to the SQLite database
def connect_db():
    conn = sqlite3.connect('library.db')
    conn.row_factory = sqlite3.Row
    return conn

# Add a new book record
@app.route('/books', methods=['POST'])
def add_book():
    data = request.get_json()
    title = data['title']
    author = data['author']
    published_date = data['published_date']
    isbn = data['isbn']

    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('INSERT INTO books (title, author, published_date, isbn) VALUES (?, ?, ?, ?)',
                   (title, author, published_date, isbn))
    conn.commit()
    conn.close()

    return jsonify({'message': 'Book added successfully'}), 201

# Retrieve a list of all books
@app.route('/books', methods=['GET'])
def get_books():
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM books')
    books = cursor.fetchall()
    conn.close()

    return jsonify([dict(row) for row in books]), 200

# Update an existing book record
@app.route('/books/<int:book_id>', methods=['PUT'])
def update_book(book_id):
    data = request.get_json()
    title = data['title']
    author = data['author']
    published_date = data['published_date']
    isbn = data['isbn']

    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('''
    UPDATE books
    SET title = ?, author = ?, published_date = ?, isbn = ?
    WHERE id = ?
    ''', (title, author, published_date, isbn, book_id))
    conn.commit()
    conn.close()

    return jsonify({'message': 'Book updated successfully'}), 200

# Delete a book record
@app.route('/books/<int:book_id>', methods=['DELETE'])
def delete_book(book_id):
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute('DELETE FROM books WHERE id = ?', (book_id,))
    conn.commit()
    conn.close()

    return jsonify({'message': 'Book deleted successfully'}), 200

if __name__ == '__main__':
    app.run(debug=True)
    
pip install -r requirements.txt

python init_db.py

python app.py

