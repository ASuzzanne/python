def create_ticket(self, project_id, violations, owner, participants):
    print("creating ticket...")

    # Format each violation detail
    violation_list_str = "\n\n".join([
        f"""**ID:** {v.get("id")}
**Parent:** {v.get("parent")}
**Name:** {v.get("name")}
**First Identified:** {v.get("first_identified")}
**OverDue:** {v.get("overdue")}
**Violation:** {v.get("violation")}"""
        for v in violations
    ])

    ticket_details = {
        "summary": f"test:computeInstance ImageOlderThanAllowed",
        "description": f"""Hi, you will soon have computeInstance ImageOlderThanAllowed violation on **{project_id}** project.

Details:

{violation_list_str}""",
        "customfield_36100": project_id,  # project key, mandatory
        "customfield_31900": "P3",        # priority, mandatory
    }

    ticket_id = raise_ticket(ticket_details, owner, participants)
    print(str(ticket_id))
    return ticket_id
# python
basics of python
def process_violations(self, project_id, violations):
    print("processing computeInstance ImageOlderThanAllowed violation...")
    violation_name = "computeInstance ImageOlderThanAllowed"
    v_ids = []

    # Check once if any violation already exists
    existing_ticket_check = any(
        ticket_exists_in_bq(violation_name, project_id, v["violation_id"])
        for v in violations
    )

    if existing_ticket_check:
        print(f"Ticket already exists for project {project_id} and violation {violation_name}")
        return

    # No ticket exists, proceed
    owner, participants = get_project_owners(project_id, PROJECT_OWNERS)
    ticket_id = self.create_ticket(project_id, violations, owner, participants)

    for violation in violations:
        v_ids.append({
            "violation_id": violation["violation_id"],
            "project_id": project_id,
            "sreops_ticket_id": ticket_id,
            "added_on": datetime.now().strftime('%Y-%m-%d %H:%M:%S.%f'),
            "owner": owner,
            "violation_name": violation_name
        })

    insert_rows_into_bq(v_ids)
class computeInstanceImageOlderThanAllowed:

    def __init__(self, project_id):
        self.project_id = project_id
        print("Successfully initialized")

    def get_details(self):
        try:
            request = compute_v1.AggregatedListInstancesRequest(project=self.project_id)
            response = instance_client.aggregated_list(request=request)

            found_instances = False
            details = []

            for zone, instances_scoped_list in response:
                if instances_scoped_list.instances:
                    found_instances = True
                    zone_clean = zone.split('/')[-1]
                    region = zone_clean.rsplit('-', 1)[0]

                    for instance in instances_scoped_list.instances:
                        instance_data = {
                            "instance_name": instance.name,
                            "instance_id": str(instance.id),
                            "instance_creation_time": instance.creation_timestamp,
                            "disks": []
                        }

                        print("Instance name:", instance.name)
                        print("Instance id:", str(instance.id))
                        print("Create time:", instance.creation_timestamp)

                        for disk in instance.disks:
                            if disk.boot and disk.source:
                                try:
                                    disk_name = disk.source.split('/')[-1]
                                    if '/regions/' in disk.source:
                                        region = disk.source.split('/regions/')[-1].split('/')[0]
                                        disk_info = regional_disk_client.get(
                                            project=self.project_id,
                                            disk=disk_name
                                        )
                                    else:
                                        disk_info = disk_client.get(
                                            project=self.project_id,
                                            zone=zone_clean,
                                            disk=disk_name
                                        )

                                    source_image = getattr(disk_info, "source_image", None)
                                    if not source_image:
                                        continue

                                    image_name = source_image.split('/')[-1]
                                    image_info = image_client.get(
                                        project=self.project_id,
                                        image=image_name
                                    )

                                    disk_data = {
                                        "disk_name": disk.device_name,
                                        "disk_creation_time": disk_info.creation_timestamp,
                                        "image_name": image_name,
                                        "image_id": str(image_info.id),
                                        "image_creation_time": image_info.creation_timestamp
                                    }

                                    instance_data["disks"].append(disk_data)

                                except Exception as e:
                                    print("Error fetching disk or image info:", str(e))

                        details.append(instance_data)

            if not found_instances:
                print("No instances found")
                return []

            return details

        except Exception as e:
            print("Error fetching instance details:", str(e))
            return []
