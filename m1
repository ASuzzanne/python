from datetime import datetime
import json
from config import *
from common import *
from computeInstanceOlder import *

def main():
    project_id = "hsbc-6320774-opschatbot-dev"

    checker = computeInstanceImageOlderThanAllowed(project_id)
    details = checker.get_details()

    print("executed successfully")

    if not details:
        print("No instances to process. Exiting.")
    else:
        violations = []
        violation_details = []

        # Print the instance details first
        print("\nOther instance details:")
        print(json.dumps(details, indent=4))

        # Now process violations
        for instance in details:
            if check_age(instance["instance_creation_time"], INSTANCE_AGE_LIMIT, "instance"):
                instance_id = instance["instance_id"]

                for disk in instance["disks"]:
                    if check_age(disk["image_creation_time"], IMAGE_AGE_LIMIT, "image"):
                        violation_id = f"{instance_id}_{disk['image_id']}"
                        violations.append(violation_id)
                        violation_data = {
                            "violation_id": violation_id,
                            "instance_name": instance["instance_name"],
                            "instance_id": instance["instance_id"],
                            "image_name": disk["image_name"],
                            "image_id": disk["image_id"]
                        }
                        violation_details.append(violation_data)

        # Printing violation IDs
        if violations:
            print("\nViolations found:", violations)

            for violation in violation_details:
                print(f"\nViolation details: {{")
                print(f"violation id: {violation['violation_id']}")
                print(f"instance name: {violation['instance_name']}, instance id: {violation['instance_id']}")
                print(f"image name: {violation['image_name']}, image id: {violation['image_id']}")
                print(f"}}")
        else:
            print("No violations found.")
