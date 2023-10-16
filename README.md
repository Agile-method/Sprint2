# Repository Name Sprint2
# Organization Name Agile-method
# Deep Panwala
# Purvesh Kapadiya

import re
from datetime import datetime
from prettytable import PrettyTable
import unittest

# GEDCOM parsing
individual_pattern = re.compile(r'0 @I(\d+)@ INDI')
name_pattern = re.compile(r'1 NAME (.+)')
gender_pattern = re.compile(r'1 SEX ([MF])')
birthday_pattern = re.compile(r'1 BIRT\n2 DATE (\d{1,2} [A-Z]{3} \d{4})')
death_pattern = re.compile(r'1 DEAT\n2 DATE (\d{1,2} [A-Z]{3} \d{4})')
family_child_pattern = re.compile(r'1 FAMC @F(\d+)@')
family_spouse_pattern = re.compile(r'1 FAMS @F(\d+)@')

family_pattern = re.compile(r'0 @F(\d+)@ FAM')
marriage_pattern = re.compile(r'1 MARR\n2 DATE (\d{1,2} [A-Z]{3} \d{4})')
divorce_pattern = re.compile(r'1 DIV\n2 DATE (\d{1,2} [A-Z]{3} \d{4})')
husband_pattern = re.compile(r'1 HUSB @I(\d+)@')
wife_pattern = re.compile(r'1 WIFE @I(\d+)@')
child_pattern = re.compile(r'1 CHIL @I(\d+)@')

individuals = {}
families = {}
current_individual = None
current_family = None

# function to convert GEDCOM date to datetime
def parse_date(date_str):
    date_format = "%d %b %Y"
    try:
        return datetime.strptime(date_str, date_format)
    except ValueError:
        return None

# User Story US08 - Birth before marriage of parents
class TestUS08BirthBeforeMarriage(unittest.TestCase):
    def test_birth_before_marriage(self):
        # Implement test cases to verify that children are born after parents' marriage.
        for family_id, family in families.items():
            marriage_date = parse_date(family['marriage_date'])
            for child_id in family['children']:
                child_birthday = parse_date(individuals[child_id]['birthday'])
                self.assertIsNotNone(child_birthday)
                self.assertIsNotNone(marriage_date)
                self.assertTrue(child_birthday >= marriage_date)

# User Story US15 - Fewer than 15 siblings
class TestUS15FewerThan15Siblings(unittest.TestCase):
    def test_fewer_than_15_siblings(self):
        # Implement test cases to verify that a family has fewer than 15 children.
        for family_id, family in families.items():
            self.assertTrue(len(family['children']) < 15)

# User Story US18 - Siblings should not marry
class TestUS18SiblingsShouldNotMarry(unittest.TestCase):
    def test_siblings_not_marry(self):
        # Implement test cases to verify that siblings do not marry each other.
        for family_id, family in families.items():
            husband_id = family['husband_id']
            wife_id = family['wife_id']
            self.assertNotEqual(individuals[husband_id]['name'], individuals[wife_id]['name'])

# User Story US21 - Correct gender for role
class TestUS21CorrectGenderForRole(unittest.TestCase):
    def test_correct_gender_for_role(self):
        # Implement test cases to verify that the gender for the husband and wife roles is correct.
        for family_id, family in families.items():
            husband_id = family['husband_id']
            wife_id = family['wife_id']
            self.assertEqual(individuals[husband_id]['gender'], 'Male')
            self.assertEqual(individuals[wife_id]['gender'], 'Female')

# Function to calculate age from birthdate
def calculate_age(birthdate):
    birthdate = parse_date(birthdate)
    if birthdate:
        end_date = datetime(2030, 9, 12)
        age = end_date.year - birthdate.year - ((end_date.month, end_date.day) < (birthdate.month, birthdate.day))
        return age
    else:
        return None




# Read and parse GEDCOM data
with open("C:\export-Forest.ged", 'r') as gedcom_file:
    lines = gedcom_file.readlines()

for line in lines:
    individual_match = individual_pattern.match(line)
    if individual_match:
        if current_individual:
            individuals[current_individual['identifier']] = current_individual
        current_individual = {
            'identifier': individual_match.group(1),
            'name': None,
            'gender': None,
            'birthday': None,
            'age': None,
            'alive': True,
            'death': None,
            'child': [],
            'spouse': [],
        }
