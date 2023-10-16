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

        name_match = name_pattern.match(line)
    if name_match and current_individual:
        current_individual['name'] = name_match.group(1)

    gender_match = gender_pattern.match(line)
    if gender_match and current_individual:
        current_individual['gender'] = 'Male' if gender_match.group(1) == 'M' else 'Female'

    birthday_match = birthday_pattern.match(line)
    if birthday_match and current_individual:
        current_individual['birthday'] = birthday_match.group(1)
        current_individual['age'] = calculate_age(birthday_match.group(1))

    death_match = death_pattern.match(line)
    if death_match and current_individual:
        current_individual['alive'] = False
        current_individual['death'] = death_match.group(1)

    family_child_match = family_child_pattern.match(line)
    if family_child_match and current_individual:
        current_individual['child'].append(family_child_match.group(1))

    family_spouse_match = family_spouse_pattern.match(line)
    if family_spouse_match and current_individual:
        current_individual['spouse'].append(family_spouse_match.group(1))

    family_match = family_pattern.match(line)
    if family_match:
        if current_family:
            families[current_family['identifier']] = current_family
        current_family = {
            'identifier': family_match.group(1),
            'marriage_date': None,
            'divorce_date': None,
            'husband_id': None,
            'husband_name': None,
            'wife_id': None,
            'wife_name': None,
            'children': [],
        }

    marriage_match = marriage_pattern.match(line)
    if marriage_match and current_family:
        current_family['marriage_date'] = marriage_match.group(1)

    divorce_match = divorce_pattern.match(line)
    if divorce_match and current_family:
        current_family['divorce_date'] = divorce_match.group(1)

    husband_match = husband_pattern.match(line)
    if husband_match and current_family:
        current_family['husband_id'] = husband_match.group(1)
        current_family['husband_name'] = individuals.get(husband_match.group(1), {}).get('name', 'Unknown')

    wife_match = wife_pattern.match(line)
    if wife_match and current_family:
        current_family['wife_id'] = wife_match.group(1)
        current_family['wife_name'] = individuals.get(wife_match.group(1), {}).get('name', 'Unknown')

    child_match = child_pattern.match(line)
    if child_match and current_family:
        current_family['children'].append(child_match.group(1))

# Create PrettyTable for individuals
individual_table = PrettyTable()
individual_table.field_names = ["Identifier", "Name", "Gender", "Birthday", "Age", "Alive", "Death", "Child", "Spouse"]

for identifier, data in sorted(individuals.items(), key=lambda x: int(x[0])):
    individual_table.add_row([
        data['identifier'],
        data['name'],
        data['gender'],
        data['birthday'],
        data['age'],
        "Yes" if data['alive'] else "No",
        data['death'] if not data['alive'] else "",
        ", ".join(data['child']),
        ", ".join(data['spouse'])
    ])

# Create PrettyTable for families
family_table = PrettyTable()
family_table.field_names = ["Family Identifier", "Married", "Divorced", "Husband ID", "Husband Name", "Wife ID",
                            "Wife Name", "Children"]

for identifier, data in sorted(families.items(), key=lambda x: int(x[0])):
    family_table.add_row([
        data['identifier'],
        data['marriage_date'],
        data['divorce_date'],
        data['husband_id'],
        data['husband_name'],
        data['wife_id'],
        ", ".join(data['children'])
    ])

# Display tables
print("Individuals Table:")
print(individual_table)

print("\nFamilies Table:")
print(family_table)

if __name__ == '__main__':
    unittest.main()

