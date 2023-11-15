using System;
using System.Collections.Generic;
using System.IO;
using System.Text.Json;
using System.Text.Json.Serialization;
using UniversityApp.DataAccess;

namespace UniversityApp
{
    public class Person
    {
        private static int _personalNumberCounter = 0;

        public string FullName { get; set; }
        public DateTime DateOfBirth { get; set; }
        public int PersonalNumber { get; private set; }

        public Person(string fullName, DateTime dateOfBirth)
        {
            FullName = fullName;
            DateOfBirth = dateOfBirth;
            PersonalNumber = ++_personalNumberCounter;
        }
    }

    public class Student : Person
    {
        public string StudentNumber { get; set; }

        public Student(string fullName, DateTime dateOfBirth, string studentNumber)
            : base(fullName, dateOfBirth)
        {
            StudentNumber = studentNumber;
        }
    }

    public class University
    {

        private List<Student> _students;

        public University()
        {
            _students = new List<Student>();
        }

        public void AddStudent(Student student)
        {
            _students.Add(student);
        }

        public IReadOnlyCollection<Student> GetStudents() => _students.AsReadOnly();

        public void RemoveStudent(Student student)
        {
            _students.Remove(student);
        }

        public Student FindStudentByNumber(string studentNumber)
        {
            return _students.Find(s => s.StudentNumber == studentNumber);
        }

    }

    namespace DataAccess
    {
        public class StudentsRepository
        {
            private const string FileName = "students.json";

            public void SaveStudents(IReadOnlyCollection<Student> students)
            {
                string json = JsonConvert.SerializeObject(students);
                File.WriteAllText(FileName, json);
            }

            public List<Student> LoadStudents()
            {
                List<Student> students;

                if (File.Exists(FileName))
                {
                    string json = File.ReadAllText(FileName);
                    students = JsonConvert.DeserializeObject<List<Student>>(json);
                }
                else
                {
                    students = new List<Student>();
                }

                return students;
            }
        }
    }

    public class Program
    {
        static void Main(string[] args)
        {
            University university = new University();
            StudentsRepository repository = new StudentsRepository();

            // Load students from file
            List<Student> students = repository.LoadStudents();
            foreach (Student student in students)
            {
                university.AddStudent(student);
            }

            // Example usage: add a new student
            Student newStudent = new Student("John Doe", new DateTime(2000, 1, 1), "S001");
            university.AddStudent(newStudent);

            // Save students to file
            repository.SaveStudents(university.GetStudents());
        }
    }
}
