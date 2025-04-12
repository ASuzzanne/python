# python
basics of python
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
