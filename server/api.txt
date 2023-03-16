const express = require('express');
const api = express();
const cors = require('cors');
const validateDate = require('validate-date');

api.use(cors());
api.use(express.json());

const randomId = () => `${Math.floor(Math.random() * 100000)}`;

const calculateAge = birthday => {
  var ageDifMs = Date.now() - birthday.getTime();
  var ageDate = new Date(ageDifMs); // miliseconds from epoch
  return Math.abs(ageDate.getUTCFullYear() - 1970);
}

const validateInput = student => {
  if (!student.firstName || student.firstName.length < 2 || student.firstName.length > 40)
    throw new Error('Invalid first name');
  
  if (!student.lastName || student.lastName.length < 2 || student.lastName.length > 40)
    throw new Error('Invalid last name');
  
  if (!student.degree || student.degree.length < 2 || student.degree.length > 40)
    throw new Error('Invalid degree');
  
  if (!validateDate(student.dob, responseType='boolean', dateFormat='yyyy-mm-dd'))
    throw new Error('Invalid date of birth (format must be yyyy-mm-dd)');

  const age = calculateAge(new Date(student.dob));
  if (age < 18 || age > 29)
    throw new Error('Student must be aged between 18 and 29');
};

const problemDetails = error => (
  {
    "type": `//invalid-student-input`,
    "title": "Invalid student input", 
    "detail": error,
    "status": 400
  }
);

let students = [
  { id: randomId(), firstName: 'John', lastName: 'Smith', dob: '1998-04-11', degree: 'Business' },
  { id: randomId(), firstName: 'Janice', lastName: 'Hope', dob: '1999-01-31', degree: 'Maths' },
  { id: randomId(), firstName: 'Douglas', lastName: 'Jones', dob: '1997-07-02', degree: 'Economics' },
  { id: randomId(), firstName: 'Wendy', lastName: 'Smith', dob: '1998-06-18', degree: 'Computer Science' },
  { id: randomId(), firstName: 'William', lastName: 'Smart', dob: '1998-12-29', degree: 'Maths' },
];

api.get('/api/students', (req, res) => {
  res.json(students);
});

api.get('/api/students/:id', (req, res) => {
  const id = req.params.id;

  const student = students.find(_ => _.id == id);
  if (!student) {
    res.status(404);
    res.send();
  } else {
    res.json(student);
  }
});

api.post('/api/students', (req, res) => {
  const student = { ...req.body, id: randomId()};

  try {
    validateInput(student);
  } catch (e) {
    res.status(400);
    res.json(problemDetails(e.message));
    return;
  }

  res.status(201);
  students.push(student);
  res.json(student);
});

api.put('/api/students/:id', (req, res) => {
  const id = req.params.id;

  if (!students.find(_ => _.id == id)) {
    res.status(404);
    res.send();
  } else {
    const student = { ...req.body, id};

    try {
      validateInput(student);
    } catch (e) {
      res.status(400);
      res.json(problemDetails(e.message));
      return;
    }

    students = [...students.filter(_ => _.id != id), student];
    res.status(204);
    res.send();
  }
});

api.delete('/api/students/:id', (req, res) => {
  const id = req.params.id;

  if (!students.find(_ => _.id == id)) {
    res.status(404);
  } else {
    res.status(204);
    students = students.filter(_ => _.id != id);
  }

  res.send();
});

api.use('/docs', express.static('docs'));

const port = 3051;

api.listen(port, () => {
  console.log(`API listening at: http://localhost:${port}`);
  console.log(`View API documentation at: http://localhost:${port}/docs`);
});