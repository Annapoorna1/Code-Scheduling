import sys

class CourseOffering:
    def __init__(self, course_name, instructor, date, min_employees, max_employees):
        self.course_name = course_name
        self.instructor = instructor
        self.date = date
        self.min_employees = min_employees
        self.max_employees = max_employees
        self.course_id = f"OFFERING-{course_name}-{instructor}"
        self.registrations = []

    def register(self, email):
        if len(self.registrations) >= self.max_employees:
            return f"COURSE_FULL_ERROR"
        registration_id = f"REG-COURSE-{email.split('@')[0]}-{self.course_name}"
        self.registrations.append(registration_id)
        if len(self.registrations) >= self.min_employees:
            return registration_id, "ACCEPTED"
        return registration_id, ""

    def cancel_registration(self, registration_id):
        if registration_id in self.registrations:
            self.registrations.remove(registration_id)
            return registration_id, "CANCEL_ACCEPTED"
        else:
            return registration_id, "CANCEL_REJECTED"

    def allot_course(self):
        status = "COURSE_CANCELED" if len(self.registrations) < self.min_employees else "CONFIRMED"
        registrations = []
        for registration_id in sorted(self.registrations):
            email = registration_id.split('-')[2]
            registrations.append((registration_id, email, self.course_id, self.course_name, self.instructor, self.date, status))
        return registrations

class CourseScheduler:
    def __init__(self):
        self.course_offerings = {}

    def add_course_offering(self, course_name, instructor, date, min_employees, max_employees):
        course_offering = CourseOffering(course_name, instructor, date, min_employees, max_employees)
        self.course_offerings[course_offering.course_id] = course_offering
        return course_offering.course_id

    def register_for_course(self, email, course_offering_id):
        if course_offering_id not in self.course_offerings:
            return "INPUT_DATA_ERROR"
        course_offering = self.course_offerings[course_offering_id]
        return course_offering.register(email)

    def cancel_registration(self, registration_id):
        for course_offering in self.course_offerings.values():
            result = course_offering.cancel_registration(registration_id)
            if result[1] == "CANCEL_ACCEPTED":
                return result
        return registration_id, "CANCEL_REJECTED"

    def allot_courses(self, course_offering_id):
        if course_offering_id not in self.course_offerings:
            return "INPUT_DATA_ERROR"
        course_offering = self.course_offerings[course_offering_id]
        return course_offering.allot_course()

def parse_command(command):
    parts = command.split(' ')
    if parts[0] == "ADD-COURSE-OFFERING":
        return ("ADD-COURSE-OFFERING", parts[1], parts[2], parts[3], int(parts[4]), int(parts[5]))
    elif parts[0] == "REGISTER":
        return ("REGISTER", parts[1], parts[2])
    elif parts[0] == "CANCEL":
        return ("CANCEL", parts[1])
    elif parts[0] == "ALLOT-COURSE":
        return ("ALLOT-COURSE", parts[1])
    else:
        return None

def process_commands(file_path):
   
